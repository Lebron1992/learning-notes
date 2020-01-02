> 这篇文章是我阅读[raywenderlich.com](https://store.raywenderlich.com)的[Design Patterns by Tutorials](https://store.raywenderlich.com/products/design-patterns-by-tutorials)的总结，文中的代码是我阅读书本之后根据自己的想法修改的。如果想看原版书籍，请点击链接购买。

***

工厂模式可以在不需要暴露创建逻辑的情况下，创建对象。主要有两部分组成：

- **Factory**：负责创建对象。
- **Products**：Factory创建的对象。

### 什么时候使用

当想要把Product的创建逻辑独立出来，而不是让使用者直接去创建时，使用这种模式。

### 简单Demo 

假设我们在开发一个HR专用的邮箱，其中有一个需求是回复求职者的职位申请时，可以根据求职者目前的状态来创建模板邮件：

首先我们有两个模型，求职者`JobApplicant`和邮件`Email`:

```swift
struct JobApplicant {
    let name: String
    let email: String
    var status: Status

    enum Status {
        case new
        case interview
        case hired
        case rejected
    }
}

struct Email {
    let subject: String
    let messageBody: String
    let recipientEmail: String
    let senderEmail: String
}
```

然后是我们的邮件工厂`EmailFactory`:

```swift
struct EmailFactory {

    let senderEmail: String

    func createEmail(to recipient: JobApplicant, messageBody: String? = nil) -> Email {
        switch recipient.status {
        case .new:
            return Email(
                subject: "已收到你的求职申请",
                messageBody: messageBody ?? "感谢你申请我们的职位，我们会在24小时内回复你。",
                recipientEmail: recipient.email,
                senderEmail: senderEmail)

        case .interview:
            return Email(
                subject: "面试邀请",
                messageBody: messageBody ?? "你的简历已经通过筛选，请于明天上午10点到我们公司面试。",
                recipientEmail: recipient.email,
                senderEmail: senderEmail)

        case .hired:
            return Email(
                subject: "你已通过面试",
                messageBody: messageBody ?? "恭喜你，你已经通过我们公司的面试，请于下周一到我们公司报道。",
                recipientEmail: recipient.email,
                senderEmail: senderEmail)

        case .rejected:
            return Email(
                subject: "面试未通过",
                messageBody: messageBody ?? "因不符合我公司的要求，此次面试不通过。谢谢！",
                recipientEmail: recipient.email,
                senderEmail: senderEmail)
        }
    }
}
```

因为发邮件的时候，需要一个发件人，所以创建邮件工厂时，需要一个`senderEmail`参数；在`createEmail`根据求职者的不同状态来创建模板，并且还提供了一个可选的`messageBody`参数，如果不提供`messageBody`，我们就会使用默认的。

使用：

```swift
var lebron = JobApplicant(name: "Lebron James",
                          email: "lebronjames@example.com",
                          status: .hired)

let emailFactory = EmailFactory(senderEmail: "hr@company.com")
let emial = emailFactory.createEmail(to: lebron)
```

### 总结

当想要把实例的创建逻辑独立出来，可以使用工厂模式。但如果想要的实例非常简单，直接在用到的地方直接创建即可。如果这个实例需要一系列的步骤才能创建，最好使用Builder模式。
