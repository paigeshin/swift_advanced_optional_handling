# (1) Being lazy is better than being non-optionally optional

- One way of avoiding optionals for properties which values need to be created after the parent object is created (such as views in a view controller — which should be created in loadView() or viewDidLoad()) is through the use of lazy properties. A lazy property can be non-optional, while still not being required to be created in its parent’s initializer. It will simply be created when first accessed.

```swift
class TableViewController: UIViewController {
    lazy var tableView = UITableView()

    override func viewDidLoad() {
        super.viewDidLoad()
        tableView.frame = view.bounds
        view.addSubview(tableView)
    }

    func viewModelDidUpdate(_ viewModel: ViewModel) {
        tableView.reloadData()
    }
}
```

# (2) Proper dependency management is better than non-optional optionals

- Another common use of optionals is to break circular dependencies. You can sometimes get into situations where A depends on B, but B also depends on A. Like in this setup:

### Circular Dependency

```swift
class UserManager {
    private weak var commentManager: CommentManager?

    func userDidPostComment(_ comment: Comment) {
        user.totalNumberOfComments += 1
    }

    func logOutCurrentUser() {
        user.logOut()
        commentManager?.clearCache()
    }
}

class CommentManager {
    private weak var userManager: UserManager?

    func composer(_ composer: CommentComposer
                  didPostComment comment: Comment) {
        userManager?.userDidPostComment(comment)
        handle(comment)
    }

    func clearCache() {
        cache.clear()
    }
}
```

### Create Composer to fix circular dependency  

```swift
class CommentComposer {
    private let commentManager: CommentManager
    private let userManager: UserManager
    private lazy var textView = UITextView()

    init(commentManager: CommentManager,
         userManager: UserManager) {
        self.commentManager = commentManager
        self.userManager = userManager
    }

    func postComment() {
        let comment = Comment(text: textView.text)
        commentManager.handle(comment)
        userManager.userDidPostComment(comment)
    }
}

class UserManager {
    private let commentManager: CommentManager

    init(commentManager: CommentManager) {
        self.commentManager = commentManager
    }

    func userDidPostComment(_ comment: Comment) {
        user.totalNumberOfComments += 1
    }
}
```

# (3) Crashing gracefully

```swift
guard let configuration = loadConfiguration() else {
    preconditionFailure("Configuration couldn't be loaded. " +
                        "Verify that Config.JSON is valid.")
}
```

# (4) Introducing Require

https://github.com/JohnSundell/Require

```swift
let configuration = loadConfiguration().require(hint: "Verify that Config.JSON is valid")
```
