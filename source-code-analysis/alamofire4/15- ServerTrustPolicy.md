这个文件主要写了如何处理服务器和客户端之间的验证。

### 1. `ServerTrustPolicyManager`

在开发过程中，我们可能会请求不同的主机地址，而一个服务器对应一个信任策略，即`ServerTrustPolicy`，所以需要一个manager来管理。

```swift
open class ServerTrustPolicyManager {
    // 信任策略数组
    open let policies: [String: ServerTrustPolicy]
    
    初始化方法
    public init(policies: [String: ServerTrustPolicy]) {
        self.policies = policies
    }

    // 根据主机地址返回对应的信任策略
    open func serverTrustPolicy(forHost host: String) -> ServerTrustPolicy? {
        return policies[host]
    }
}
```

给`URLSession`添加`serverTrustPolicyManager`。

```swift
extension URLSession {
    private struct AssociatedKeys {
        static var managerKey = "URLSession.ServerTrustPolicyManager"
    }

    var serverTrustPolicyManager: ServerTrustPolicyManager? {
        get {
            return objc_getAssociatedObject(self, &AssociatedKeys.managerKey) as? ServerTrustPolicyManager
        }
        set (manager) {
            objc_setAssociatedObject(self, &AssociatedKeys.managerKey, manager, .OBJC_ASSOCIATION_RETAIN_NONATOMIC)
        }
    }
}
```

### 2. `ServerTrustPolicy`

在通过HTTPS连接服务器时，如果需要认证，`ServerTrustPolicy`就会评估服务器信任，最终决定服务器信任是否有效和是否建立连接。

#### 1) 提供了6种评估机制

```swift
// 执行默认的评估
case performDefaultEvaluation(validateHost: Bool)

// 执行撤销评估
case performRevokedEvaluation(validateHost: Bool, revocationFlags: CFOptionFlags)

// 使用证书来验证服务器信任
case pinCertificates(certificates: [SecCertificate], validateCertificateChain: Bool, validateHost: Bool)

// 使用公钥来验证服务器信任
case pinPublicKeys(publicKeys: [SecKey], validateCertificateChain: Bool, validateHost: Bool)

// 禁用评估
case disableEvaluation

// 自定义评估
case customEvaluation((_ serverTrust: SecTrust, _ host: String) -> Bool)
```

#### 2) 获取证书和公钥

```swift
// 获取证书
public static func certificates(in bundle: Bundle = Bundle.main) -> [SecCertificate] {
    var certificates: [SecCertificate] = []

    let paths = Set([".cer", ".CER", ".crt", ".CRT", ".der", ".DER"].map { fileExtension in
        bundle.paths(forResourcesOfType: fileExtension, inDirectory: nil)
        }.joined())

    for path in paths {
        if
            let certificateData = try? Data(contentsOf: URL(fileURLWithPath: path)) as CFData,
            let certificate = SecCertificateCreateWithData(nil, certificateData)
        {
            certificates.append(certificate)
        }
    }

    return certificates
}

// 从所有证书中取出所有公钥
public static func publicKeys(in bundle: Bundle = Bundle.main) -> [SecKey] {
    var publicKeys: [SecKey] = []

    for certificate in certificates(in: bundle) {
        if let publicKey = publicKey(for: certificate) {
            publicKeys.append(publicKey)
        }
    }

    return publicKeys
}
```

#### 3) 评估服务器信任

对于需要验证的评估机制，总体思路如下：1）使用`SecPolicyCreateSSL`创建`SecPolicy`; 2) 使用`SecTrustSetPolicies`为`SecTrust`设置安全策略; 3) 验证是否有效

```swift
// 评估服务器信任是否有效
public func evaluate(_ serverTrust: SecTrust, forHost host: String) -> Bool {
    var serverTrustIsValid = false

    switch self {
    case let .performDefaultEvaluation(validateHost):
        let policy = SecPolicyCreateSSL(true, validateHost ? host as CFString : nil)
        SecTrustSetPolicies(serverTrust, policy)

        serverTrustIsValid = trustIsValid(serverTrust)
    case let .performRevokedEvaluation(validateHost, revocationFlags):
        let defaultPolicy = SecPolicyCreateSSL(true, validateHost ? host as CFString : nil)
        let revokedPolicy = SecPolicyCreateRevocation(revocationFlags)
        SecTrustSetPolicies(serverTrust, [defaultPolicy, revokedPolicy] as CFTypeRef)

        serverTrustIsValid = trustIsValid(serverTrust)
    case let .pinCertificates(pinnedCertificates, validateCertificateChain, validateHost):
        if validateCertificateChain { // 需要认证证书链
            let policy = SecPolicyCreateSSL(true, validateHost ? host as CFString : nil)
            SecTrustSetPolicies(serverTrust, policy)

            SecTrustSetAnchorCertificates(serverTrust, pinnedCertificates as CFArray)
            SecTrustSetAnchorCertificatesOnly(serverTrust, true)

            serverTrustIsValid = trustIsValid(serverTrust)
        } else {  // 不需要认证证书链
            let serverCertificatesDataArray = certificateData(for: serverTrust)
            let pinnedCertificatesDataArray = certificateData(for: pinnedCertificates)

            // outerLoop：给外面这个for循环设置一个名字
            outerLoop: for serverCertificateData in serverCertificatesDataArray {
                for pinnedCertificateData in pinnedCertificatesDataArray {
                    if serverCertificateData == pinnedCertificateData {
                        // 只要服务器信任的证书和指定的证书有一个匹配，就验证通过
                        serverTrustIsValid = true
                        break outerLoop
                    }
                }
            }
        }
    case let .pinPublicKeys(pinnedPublicKeys, validateCertificateChain, validateHost):
        var certificateChainEvaluationPassed = true

        if validateCertificateChain { // 需要认证证书链
            let policy = SecPolicyCreateSSL(true, validateHost ? host as CFString : nil)
            SecTrustSetPolicies(serverTrust, policy)

            certificateChainEvaluationPassed = trustIsValid(serverTrust)
        }

        if certificateChainEvaluationPassed { // 通过了证书链认证
            outerLoop: for serverPublicKey in ServerTrustPolicy.publicKeys(for: serverTrust) as [AnyObject] {
                for pinnedPublicKey in pinnedPublicKeys as [AnyObject] {
                    if serverPublicKey.isEqual(pinnedPublicKey) {
                        // 只要服务器信任的公钥和指定的公钥相同，则验证通过
                        serverTrustIsValid = true
                        break outerLoop
                    }
                }
            }
        }
    case .disableEvaluation:
        serverTrustIsValid = true
    case let .customEvaluation(closure):
        serverTrustIsValid = closure(serverTrust, host)
    }

    return serverTrustIsValid
}

// 判断服务器信任是否有效
private func trustIsValid(_ trust: SecTrust) -> Bool {
    var isValid = false

    var result = SecTrustResultType.invalid
    let status = SecTrustEvaluate(trust, &result)

    if status == errSecSuccess {
        let unspecified = SecTrustResultType.unspecified
        let proceed = SecTrustResultType.proceed

        // 如果结果是unspecified或者proceed，则认为是有效的
        isValid = result == unspecified || result == proceed
    }

    return isValid
}
```

#### 4) 其他私有的方法

```swift
// 提供一个SecTrust，返回对应的证书数据
private func certificateData(for trust: SecTrust) -> [Data] {
    var certificates: [SecCertificate] = []

    for index in 0..<SecTrustGetCertificateCount(trust) {
        if let certificate = SecTrustGetCertificateAtIndex(trust, index) {
            certificates.append(certificate)
        }
    }

    return certificateData(for: certificates)
}
// 把证书转化为数据
private func certificateData(for certificates: [SecCertificate]) -> [Data] {
    return certificates.map { SecCertificateCopyData($0) as Data }
}

// 提供一个SecTrust，返回对应的公钥
private static func publicKeys(for trust: SecTrust) -> [SecKey] {
    var publicKeys: [SecKey] = []

    for index in 0..<SecTrustGetCertificateCount(trust) {
        if
            let certificate = SecTrustGetCertificateAtIndex(trust, index),
            let publicKey = publicKey(for: certificate)
        {
            publicKeys.append(publicKey)
        }
    }

    return publicKeys
}

// 从某个证书中取出公钥
private static func publicKey(for certificate: SecCertificate) -> SecKey? {
    var publicKey: SecKey?

    let policy = SecPolicyCreateBasicX509()
    var trust: SecTrust?
    let trustCreationStatus = SecTrustCreateWithCertificates(certificate, policy, &trust)

    if let trust = trust, trustCreationStatus == errSecSuccess {
        publicKey = SecTrustCopyPublicKey(trust)
    }

    return publicKey
}
```
