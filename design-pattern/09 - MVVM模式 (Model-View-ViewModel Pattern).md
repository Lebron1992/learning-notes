> 这篇文章是我阅读[raywenderlich.com](https://store.raywenderlich.com)的[Design Patterns by Tutorials](https://store.raywenderlich.com/products/design-patterns-by-tutorials)的总结，文中的代码是我阅读书本之后根据自己的想法修改的。如果想看原版书籍，请点击链接购买。

***

MVVM是一个结构设计模式，主要有三部分组成：

- **Model**：保存应用的数据，通常是struct或者class。
- **View**：显示应用界面，通常继承自`UIView`。
- **View Model**：把Model的数据转化成可以显示在View上的数据，通常是class类型。

### 什么时候使用

当我们需要把Model的数据转化成可以显示在View的时候使用。例如，把`Date`转化成以`String`的形式显示。如果我们使用MVC的话，就需要把转化的代码写到View Controller，造成View Controller的代码太多，就成了我们所说的Massive View Controller。

### 简单Demo 

我们假设要显示一个当前用户正在关注的用户列表，使用MVVM模式。Model是`User`；View Model是`UserListViewModel`；`UserListViewController`负责管理View。

#### User

```swift
struct UsersResults: Codable {
    let users: [User]
}

struct User: Codable {
    var userID: String?
    var username: String?
}
```

`User`遵循`Codable`；`UsersResults`用于辅助解析服务器返回用户数据，同样遵循`Codable`。关于`Codable`，如果不懂的，请大家自行去学习了解。

#### UserListViewModel

```swift
final class UserListViewModel {

    private var users: [User] = []

    // MARK: - Data

    func numberOfRows() -> Int {
        return users.count
    }

    func user(at index: Int) -> User? {
        guard index < users.count else { return nil }
        return users[index]
    }

    // MARK: - Fetching

    private let decoder = JSONDecoder()

    func fetchFollowings(userID: String, completion: (() -> Void)? = nil) {
        let url = "https://api.your_app_name.com/api/v1/users/\(userID)/followings"
        RequestManager.request(.get, urlString: url) { [weak self] (result) in
            switch result {
            case .success(let data):

                guard let jsonData = data,
                    let result = try? self?.decoder.decode(UsersResults.self, from: jsonData),
                    let users = result.users else {
                        self?.users = []
                        return
                }
                self?.users = users

            case .failure(let error):
                self?.users = []
            }
        }
    }
}
```

我们上面讲到，View Model是要把Model的数据转化成可以显示在View上的数据，所以View想要什么数据，View Model就准备什么数据。

首先要从服务器获取数据，所以写了一个`fetchFollowings`方法。`RequestManager`是我自己封装的，`request`方法的最后一个参数是closure，请求完成后调用，closure里包含一个枚举类型的result: 1) `.success(Data?)`: 如果请求成功，并且有数据，那我们解析成`[User]`；2) `.fail(Error?)`: 请求失败。

要显示用户列表，通常用一个`UITableView`，我们需要告诉`UITableView`有多少行，每一行对应的`User`，所以就有了`numberOfRows()`和`user(at index: Int)`方法。

#### UserListViewController

```swift
final class UserListViewController: UIViewController {

    private let viewModel: UserListViewModel

    // MARK: - Initializers

    init(withViewModel vm: UserListViewModel) {
        viewModel = vm
        super.init(nibName: nil, bundle: nil)
    }

    required init?(coder aDecoder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }

    // MARK: - Views

    private lazy var tableView: UITableView = {
        let tv = UITableView(frame: .zero)
        tv.dataSource = self
        tv.register(UITableViewCell.self, forCellReuseIdentifier: "UserCell")
        return tv
    }()

    // MARK: - View LifeCycles

    override func loadView() {
        view = UIView()
        view.backgroundColor = .white
        addTableView()
    }

    override func viewDidLoad() {
        super.viewDidLoad()

        let userID = "xxxxxxxxxxxxxx"
        viewModel.fetchFollowings(userID: userID) { [weak self] in
            self?.tableView.reloadData()
        }
    }

    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}

// MARK: - UITableViewDataSource
extension UserListViewController: UITableViewDataSource {

    func tableView(_ tableView: UITableView,
                   numberOfRowsInSection section: Int) -> Int {
        return viewModel.numberOfRows()
    }

    func tableView(_ tableView: UITableView,
                   cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        let cell = tableView.dequeueReusableCell(withIdentifier: "UserCell", for: indexPath)
        let user = viewModel.user(at: indexPath.row)
        cell.textLabel?.text = user?.username
        return cell
    }
}

// MARK: - Setup Subviews
extension UserListViewController {
    private func addTableView() {
        view.addSubview(tableView)
        tableView.translatesAutoresizingMaskIntoConstraints = false

        NSLayoutConstraint.activate([
            tableView.topAnchor.constraint(equalTo: view.topAnchor),
            tableView.bottomAnchor.constraint(equalTo: view.bottomAnchor),
            tableView.leadingAnchor.constraint(equalTo: view.leadingAnchor),
            tableView.trailingAnchor.constraint(equalTo: view.trailingAnchor)
            ])
    }
}
```

在MVVM模式中，我们会让View Controller持有一个View Model，通过初始化函数完成初始化；在`loadView()`添加`UITableView`；在实现`UITableViewDataSource`中，我们直接通过View Model来告诉tableview所需要的数据；在`viewDidLoad`中，通过View Model获取数据，请求完成后，刷新tableview。

### 总结

使用MVVM模式，我们把大量的代码转移到了View Model，大大减轻了View Controller的负担。View Controller需要什么数据，就让View Model告诉它。非常适合在比较大型的应用中使用。如果是第一次学习应用开发，建议还是从MVC开始。
