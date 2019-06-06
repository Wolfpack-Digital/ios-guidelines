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
        - `Models` - business layer models
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


9. When fetching data from an API use the APIClient - Factory - Model convention

Example (read comments as well):

    //APIClient - used for fetching the raw info from the API
    class APIClient {
        class func getRequests(completion: @escaping (_ requests: [Request]) -> ()) {
            get(path: "requests") { json in
                if let jsonArray = json as? [[String: AnyObject]] {   
                    //Passes the info to the Factory and expects it modeled
                    let requests = Factory.requestsFromJsonArray(jsonArray: jsonArray)
                    completion(requests)
                }
            }
        }
    }
    
    //Works like a classic factory: raw material goes in -> processed material comes out
    class Factory {
        class func requestsFromJsonArray(jsonArray: [[String: AnyObject]]) -> [Request] {
            var requests = [Request]()
            for json in jsonArray {
                requests += [requestFromJson(json: json)]
            }
            
            return requests
        }
        
        class func requestFromJson(json: [String: AnyObject]) -> Request {
            let dateFormatter = DateFormatter()
            dateFormatter.timeZone = TimeZone(identifier: "UTC")
            dateFormatter.dateFormat = "yyyy-MM-dd'T'HH:mm:ss.SSSZ"
            
            var departureDate, arrivalDate, createdAt: Date?
            
            if let departureDateString = json["departure_date"] as? String {
                departureDate = dateFormatter.date(from: departureDateString)
            }
            
            if let arrivalDateString = json["arrival_date"] as? String {
                arrivalDate = dateFormatter.date(from: arrivalDateString)
            }
            
            if let createdAtString = json["created_at"] as? String {
                createdAt = dateFormatter.date(from: createdAtString)
            }
            
            return Request(
                isPlanned: json["is_planned"] as? Bool,
                departureStation: json["departure_station"] as? String,
                arrivalStation: json["arrival_station"] as? String,
                departureTime: departureDate,
                arrivalTime: arrivalDate,
                state: RequestState(rawValue: json["state"] as! String),
                assistanceTypes: json["assistance_types"] as? [String],
                createdAt: createdAt
            )
        }
    }
    
    //Model: used for passing around information.
    enum RequestState : String {
        case pending = "pending"
        case assistantOnTheWay = "assistant_on_the_way"
        case attended = "attended"
        case confirmed = "confirmed"
    }
    
    class Request: NSObject {
        var isPlanned: Bool?
        var state: RequestState?
        var assistanceTypes: [String]?
        var departureStation: String?
        var arrivalStation: String?
        var departureTime: Date?
        var arrivalTime: Date?
        var message: String?
        var createdAt: Date?
        
        init(
            isPlanned: Bool?,
            departureStation: String?,
            arrivalStation: String?,
            departureTime: Date?,
            arrivalTime: Date?,
            state: RequestState?,
            assistanceTypes: [String]?,
            createdAt: Date?
        ) {
            self.isPlanned = isPlanned
            self.departureStation = departureStation
            self.arrivalStation = arrivalStation
            self.departureTime = departureTime
            self.arrivalTime = arrivalTime
            self.state = state
            self.assistanceTypes = assistanceTypes
            self.createdAt = createdAt
        }
    }



10. Split APIClient into multiple extensions based on the model

Example:

The main APIClient class - only contains helper methods, works like a wrapper for Alamofire in this case:

    class APIClient {
        
        static func get(path: String, params: [String: Any]?=nil, completion: @escaping (_ json: Any?) -> ()) {
            performRequest(path: path, method: .get, params: params, completion: completion)
        }
        
        static func post(path: String, params: [String: Any]?=nil, completion: @escaping (_ json: Any?) -> ()) {
            performRequest(path: path, method: .post, params: params, completion: completion)
        }
        
        static func patch(path: String, params: [String: Any]?=nil, completion: @escaping (_ json: Any?) -> ()) {
            performRequest(path: path, method: .patch, params: params, completion: completion)
        }
        
        // Private
        
        private static func performRequest(path: String, method: HTTPMethod, params: [String: Any]?, completion: @escaping (_ json: Any?) -> ()) {
            var requestParams = [String: Any]()
            
            if let p = params {
                requestParams = p
            }
            
            if let deviceToken = SessionManager.deviceToken() {
                requestParams["device_token"] = deviceToken
            }
    
            Alamofire.request(urlWithPath(path), method: method, parameters: requestParams).validate().responseJSON {
                response in
                
                completion(response.result.value)
            }
        }
        
        private static func urlWithPath(_ string: String) -> String {
            var baseUrl: String!
            
            switch serverEnv() {
            case .staging:
                baseUrl = stagingUrl()
            case .local:
                baseUrl = localUrl()
            case .production:
                baseUrl = productionUrl()
            }
            
            return baseUrl + string
        }
        
        private enum ServerEnv {
            case staging
            case production
            case local
        }
        
        private static func serverEnv() -> ServerEnv {
            return .staging
        }
        
        private static func stagingUrl() -> String {
            return "https://myawesomeapi.com/api/v1/"
        }
        
        private static func localUrl() -> String {
            return "http://192.168.0.41.xip.io/api/v1/"
        }
        
        private static func productionUrl() -> String {
            return "https://myawesomeproductionapi/api/v1/"
        }
    }

Extension for Profile API calls:

    //APIClientProfile.swift
    
    extension APIClient {
        
        static func createProfile(profile: Profile, completion: @escaping (_ succeeded: Bool, _ profile: Profile?) -> ()) {
            post(path: "profiles", params: profileParams(profile: profile)) { json in
                if let profileJson = json as? [String : AnyObject] {
                    let profile = Factory.profileFromJson(json: profileJson)
                    completion(true, profile)
                } else {
                    completion(false, nil)
                }
            }
        }
        
        class func updateProfile(profile: Profile, completion: @escaping (_ succeeded: Bool, _ profile: Profile?) -> ()) {
            post(path: "profiles/update_profile", params: profileParams(profile: profile)) { json in
                if let profileJson = json as? [String : AnyObject] {
                    let profile = Factory.profileFromJson(json: profileJson)
                    completion(true, profile)
                } else {
                    completion(false, nil)
                }
            }
        }
        
        class func getProfile(completion: @escaping (_ profile: Profile) -> ()) {
            get(path: "profiles/\(SessionManager.deviceToken()!)") { json in
                if let profileJson = json as? [String: AnyObject] {
                    let profile = Factory.profileFromJson(json: profileJson)
                    completion(profile)
                }
            }
        }
        
        private class func profileParams(profile: Profile) -> [String : Any] {
            var params = [
                "device_token" : SessionManager.deviceToken()!
            ] as [String : Any]
            
            if let assistanceTypes = profile.assistanceTypes {
                params["profile[assistance_types]"] = assistanceTypes
            }
            
            if let name = profile.name {
                params["profile[name]"] = name
            }
            
            if let phone = profile.phoneNumber {
                params["profile[phone]"] = phone
            }
            
            return params
        }
        
    }

Extension for Devices API calls:


    //APIClientDevices.swift
    
    extension APIClient {
        
        static func postDevice(_ completion: @escaping (_ deviceToken: String, _ succeeded: Bool) -> ()) {
            post(path: "devices", params: ["device[os]": "iOS"]) { json in
                if let jsonDict = json as? [String:AnyObject] {
                    if jsonDict["token"] != nil {
                        completion(jsonDict["token"] as! String, true)
                    } else {
                        completion("", false)
                    }
                } else {
                    completion("", false)
                }
            }
        }
        
        static func postPushNotificationsToken(devicePNToken: String, _ completion: @escaping (_ succeeded: Bool) -> ()) {
            let params = [
                "device[push_notification_token]": devicePNToken
            ]
            
            patch(path: "devices/\(SessionManager.deviceToken()!)", params: params) { json in
                if let jsonDict = json as? [String:AnyObject] {
                    completion(jsonDict["push_notification_token"] != nil)
                } else {
                    completion(false)
                }
            }
        }
        
    }

And so on … You get the point 😃 

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
