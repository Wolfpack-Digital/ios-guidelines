# iOS guidelines
Guidelines on how we write and structure iOS projects at Wolfpack-Digital


1. Structure of the project folders. The structure follows some rules: high level code top -> low level code bottom. If the layer is the same, use user flow order, if user flow doesn't matter, use alpanumeric ordering. Use the example bellow as a starting point.
    e.g.:
    - `App` - for AppDelegate, extensions of AppDelegate and global clases that are supposed to be used at entry level of the app
    - `Navigation` - for coordinators and other logic related to navigation (e.g. deeplinks)
    - `Storyboards` - group each flow in his own storyboard. Avoid using segues as they are hard to be reused
    - `Presentation` - for view controllers and extensions of view controllers
        - `Launch` - for launch and app start loading UI
        - `Authentification` - Login, Signup related UI
            - `Welcome`
                - `...`
            - `Login`
                - `LoginViewController`
                    - `LoginViewController.xib` - `.xib` file above its counterpart `.swift` file
                    - `LoginViewController.swift`
                    - `LoginViewModel.swift` - viewModel bellow the view controller
                - `Cells` - cells used only for the login screen
            - `Signup`
                - `...`
        - `Main` - main screens of the app. Add more levels of subgroupping if needed.
        - `Cells` - cells supposed to be used throughout the app
        - `Views` - custom views that are used throughout the app
    - `Networking` - helpers for managing API requests. Group APIs in separate files
        - `Models` - network layer models
    - `Resources` - assets, colors, plists, fonts, etc.
    - `Utilities`
        - `Error`
        - `Extensions`
        - `Session`
        - `...`
    
    

2. Use extensions to implement UITableViewDelegate + UITableViewDataSource. It’s fine to put them in the same .swift file.

Example:
  
    class RequestsViewController: UIViewController {

        @IBOutlet weak var tableView: UITableView!
        var requests = [Request]()

        override func viewDidLoad() {
            super.viewDidLoad()      
            //do something
        }
    }

    extension RequestsViewController: UITableViewDelegate, UITableViewDataSource {
        func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
            return requests.count
        }

        func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
            //dequeue cell ...

            return cell
        }

        func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
            //do something
        }
    }
    

3. Use simple naming conventions


- If there’s only one UI element of the kind, use a simple name. For example: if your `ViewController` contains only 1 `UITableView`, use the name `tableView`
- State the type of the UI element in it’s name - this way you or other developers will know what it is when reading the code. For example: 
    - use `destinationLabel` instea of simply `destination`
    - use `containerView` instead of just `container`
- Never use super short shortcuts for variables such as `cv` for `containerView` or `dF` for `dateFormatter`


Example

    @IBOutlet weak var tableView: UITableView! //if there's only 1 tableView
    @IBOutlet weak var destinationLabel: UILabel! 
    


4. Don’t forget to call super when overriding a method

Example

    override func viewDidLoad() {
        super.viewDidLoad()
        //do your thing here :-)
    }
5. Never leave Xcode empty auto generated methods or comments in the code

Instead of this:

    class ViewController: UIViewController {
    
        override func viewDidLoad() {
            super.viewDidLoad()
            loadDataFromAPI()
            // Do any additional setup after loading the view.
        }
        
        func loadDataFromAPI() {
            //load it
        }
    
        override func didReceiveMemoryWarning() {
            super.didReceiveMemoryWarning()
            // Dispose of any resources that can be recreated.
        }
        
        /*
        // MARK: - Navigation
    
        // In a storyboard-based application, you will often want to do a little preparation before navigation
        override func prepare(for segue: UIStoryboardSegue, sender: Any?) {
            // Get the new view controller using segue.destinationViewController.
            // Pass the selected object to the new view controller.
        }
        */
    }

Do this:

    class ViewController: UIViewController {
    
        override func viewDidLoad() {
            super.viewDidLoad()
            loadDataFromAPI()
        }
        
        func loadDataFromAPI() {
            //load it
        }
    }
    
    

6. Follow the ordering convention for elements in the ViewControllers:
- IBOutlets first
- Variables
- View Cycle method overrides (viewDidLoad, viewWillAppear …)
- IBAction methods
- Public methods
- Private methods
- Extensions


7. Follow the conventions for methods:
- if the method doesn’t return anything, then the name should always represent an action. Example: `setBorderSettings` instead of `borderSettings`. You could use `borderSettings` if it returns some kind of settings.
- the name should be as descriptive as possible - longer names are better then unclear names.
- if, when setting the name, you realize you have an `and` like `updateTitleAndColor`  then you need to have 2 methods `updateTitle` and `updateColor`
- Split into multiple methods instead of putting comments


8. `viewDidLoad` or any other lifecycle overridden method should be as skinny as possible - only call other methods from there.

Example:

    class ViewController: UIViewController {
    
        override func viewDidLoad() {
            super.viewDidLoad()
            loadDataFromAPI()
            customizeUI()
            showLoadingIndicator()
        }
        
    }


9. API Client Structure

Api Client Configuration
     - Generic protocol implemented by the ApiRouter enum 

Example:

protocol APIConfiguration: URLRequestConvertible {
    var baseUrl: String { get }
    var method: HTTPMethod { get }
    var path: String { get }
    var headers: [String: String] { get }
    var parameters: Parameters? { get }
   
}

APIRouter 

- Enum that defines the  route, HTTP method, parameters and headers for each api call

Example:

enum APIRouter: APIConfiguration {

    case register(request: RegisterRequest)

    var baseUrl: TargetConfig.baseUrl

    method: HTTPMethod {
        switch self {
            case .register: return .post
        }
    }

    var path: String {
        switch self {
            case .register: “api/v1/register”
        }
    }

    var headers : [String: String] {
                var headers: [String: String] = [:]
                headers[“Authorization”] = authToken
        return headers
    }

    var parameters: Parameters? {
        switch self {
            case .register(let request):
                return request.params
        }
    }
    
    // the RegisterRequest model must implement the Encodable protocol

}

API Client
- Generic protocol user for API request/response management
- Wrapper for the Alamofire requests

Example: 

protocol APIClient : {

        func performRequest<T: Decodable>(route: APIConfiguration, decoder: JSONDecoder,
                                      completion: @escaping (APIResponse<T>) -> Void)
         func performRequest(route: APIRouter, completion: @escaping (Bool, String?) -> Void)
           func upload<T: Decodable>(route: APIRouter, decoder: JSONDecoder, completion: @escaping (APIResponse<T>) -> Void)
}

10. The API calls are performed by the Repository class that implements the APIClient protocol

APIResponse
- Generic response model used to encapsulate the request status and data
- Expected by repositories

struct APIResponse<DataModel> {
    let success: Bool
    let message: String?
    let data: DataModel?
    
    static func success<DataModel>(_ data: DataModel?) -> APIResponse<DataModel> {
        return APIResponse<DataModel>(success: true, message: nil, data: data)
    }
    
    static func failure(_ message: String?, _ data: DataModel? = nil) -> APIResponse<DataModel> {
        return APIResponse<DataModel>(success: false, message: message, data: data)
    }
    
}

Example of an API call performed in the Repository class :

func createAccount(account: AccountData, _ completion: @escaping (APIResponse<UserProfile>) -> Void) {

        performRequest(route: APIRouter.register(request: AccountRequest(account: account))) { [weak self] (response: APIResponse<UserResponse>) in
            guard let `self` = self else { return }
            if let data = response.data, response.success {
                self.fetchCategoriesAndProductsIfNeeded {
                    completion(APIResponse(success: true, message: nil, data: data.hairstylist))
                }
            } else {
                completion(APIResponse(success: false, message: response.message, data: nil))
            }
        }
    }


11. Use SwiftLint to enforce stylies and conventions

More details on how to install it: https://github.com/realm/SwiftLint


12. Always use small descriptive methods instead of big ones

This video describes this best
https://www.screenmailer.com/v/Mtu1mar72Yocj48


13. Never leave logs in your commits 

Comment them if you might need them later or they’re complex, but never leave them in. The reason behind this is that the project will clutter with logs and when you’re really debugging something you’re only looking for certain log messages, not all, so why have all that noise?


14. DRY - Do not Repeat Yourself

This is a very important guideline - it’s what marks a junor from an intermediate/senior developer.

Do not write the same logic twice - if you find yourself copy pasting things within the same project - ask yourself: can I move this in another class so that I can reuse it anywhere I need it?
