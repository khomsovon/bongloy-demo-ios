<p align="center"><img src="https://cdn.bongloy.com/assets/logos/bongloy_logo-01f89eca1fc6ec70a7d1dfd1b0e9df6429e16eb283a3f01e4b07551009f2e2ee.png" width="200"></p>

Bongloy-demo-ios is a iOS app which demonstrates how to use Bongloy.
### Screenshots
<img src="Screens/Bongloy-demo-ios.gif" width="300">

## Installation

    $ git clone https://github.com/khomsovon/bongloy-demo-ios.git
  ```sh
  cd bongloy-demo-ios
  ```
  ```sh
  pod install
  ```
  ```sh
  open bongloy-demo-ios.xcworkspace
  ```
  Fill in the `BONGLOY_PUBLISHABLE_KEY` and `BACKEND_BASE_URL` constant in ./bongloy-demo-ios/Utilities/Constants.swift
```swift
let BACKEND_BASE_URL = "Your backend base url"
let BONGLOY_PUBLISHABLE_KEY = "sk_test_****************************************************************"
```
`BONGLOY_PUBLISHABLE_KEY` Find it [here](https://sandbox.bongloy.com/dashboard/account_details)
and Bongloy demo backend [here](https://github.com/bongloy/bongloy-demo-laravel)
## Integration
#### Install and configure the SDK
   - If you haven't already, install the latest version of [CocoaPods](https://guides.cocoapods.org/using/getting-started.html)
   - Add this line to your Podfile
      ```ssh
      pod 'Stripe'
      ```
   - Run the following command
      ```ssh
      pod install
      ```
   - Don't forget to use the .xcworkspace file to open your project in Xcode, instead of the .xcodeproj file, from here on out.
   - In the future, to update to the latest version of the SDK, just run:
      ```ssh
      pod update Stripe
      ```

#### Configure your Bongloy integration in your App Delegate

After you're done installing the SDK, configure it with your Bongloy API keys in `AppDelegate.swift`.

``` swift
import UIKit
import Stripe

@UIApplicationMain
class AppDelegate: UIResponder, UIApplicationDelegate {

  func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplicationLaunchOptionsKey: Any]?) -> Bool {
      STPPaymentConfiguration.shared().publishableKey = "pk_test_2411c55a75ad6d004eaaf240f99b577dec6d6630789c06a23639967ae3c10572"
      // do any other necessary launch configuration
      return true
  }
}
```
#### Create `BongloyAPIClient` class
  ![alt text](https://cl.ly/712522276e79/download/Image%2525202018-09-24%252520at%2525205.34.16%252520PM.png)

  Make sure `Subclass of` selected STPAPIClient and Language Objective-C click Next -> Create -> Create Bridging Header

  `projectName-Bridging-Header.h`
  ``` objc
    #import "BongloyAPIClient.h"
  ```
  `BongloyAPIClient.h`
  ``` objc
  @interface BongloyAPIClient : STPAPIClient

  /**
   A shared singleton API client. Its API key will be initially equal to [Bongloy defaultPublishableKey].
   */

  + (instancetype)sharedBongloyClient;

  /**
   Initializes an API client with the given configuration. Its API key will be
   set to the configuration's publishable key.

   @param configuration The configuration to use.
   @return An instance of BongloyAPIClient.
   */

  - (instancetype)initWithConfiguration:(STPPaymentConfiguration *)configuration NS_DESIGNATED_INITIALIZER;

  @end

  @interface BongloyAPIClient()

  @property (nonatomic, strong, readwrite) NSURL *apiURL;
  @property (nonatomic, strong, readonly) NSURLSession *urlSession;

  @end
  ```
  `BongloyAPIClient.m`
  ``` objc
  #import "BongloyAPIClient.h"

  static NSString * const APIBaseURL = @"https://api.bongloy.com/v1";

  @interface BongloyAPIClient()

  @property (nonatomic, strong, readwrite) NSMutableDictionary<NSString *,NSObject *> *sourcePollers;
  @property (nonatomic, strong, readwrite) dispatch_queue_t sourcePollersQueue;
  @property (nonatomic, strong, readwrite) NSString *apiKey;

  @end

  @implementation BongloyAPIClient

  + (instancetype)sharedBongloyClient {
    static id sharedBongloyClient;
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{ sharedBongloyClient = [[self alloc] init]; });
    return sharedBongloyClient;
  }

  - (instancetype)init {
    return [self initWithConfiguration:[STPPaymentConfiguration sharedConfiguration]];
  }

  - (instancetype)initWithConfiguration:(STPPaymentConfiguration *)configuration {
    NSString *publishableKey = [configuration.publishableKey copy];
    self = [super initWithConfiguration:configuration];
    if (self) {
        _apiKey = publishableKey;
        _apiURL = [NSURL URLWithString:APIBaseURL];
        super.configuration = configuration;
        super.stripeAccount = configuration.stripeAccount;
        _sourcePollers = [NSMutableDictionary dictionary];
        _sourcePollersQueue = dispatch_queue_create("com.stripe.sourcepollers", DISPATCH_QUEUE_SERIAL);
        _urlSession = [NSURLSession sessionWithConfiguration:[NSURLSessionConfiguration defaultSessionConfiguration]];
    }
    return self;
  }
  @end
  ```
#### Create Token
  ``` swift
    let cardParams = STPCardParams()
    cardParams.number = "4242424242424242"
    cardParams.expMonth = 10
    cardParams.expYear = 2018
    cardParams.cvc = "123"

    BongloyAPIClient.sharedBongloy().createToken(withCard: cardParam) { (token: STPToken?, error: Error?) in
      guard let token = token, error == nil else {
          // Present error to user...
          return
      }

      // do anything
    }
  ```
## Official Documentation

Documentation for Bongloy can be found on the [Bongloy website](https://www.bongloy.com/documentation).
