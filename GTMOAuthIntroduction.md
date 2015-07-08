# Using the OAuth Controllers #



## Introduction ##

[OAuth 1](http://oauth.net/) is a protocol allowing your application to obtain authorization to read or modify a user’s files or data on an external server.

The server generates a web page for the user to sign in with her name and password, including a button explicitly granting access to some of her data. Upon successful authentication, the server gives a token to your application representing the user's authorization.

With the Objective-C OAuth controllers, the web page can be presented as an iOS view or a Mac sheet within your application. The controllers also provide authentication objects that simplify your application's future requests for the user's data.

OAuth 1 relies on a fragile and complex request-signing process, and so is generally being supplanted by the newer [OAuth 2](http://oauth.net/2/) standard.

## Using the iOS and Mac OS X OAuth Controllers ##

The OAuth controllers are useful for authenticating to a variety of services which require OAuth 1.

There are example iOS and Mac applications using the OAuth controllers in the library's [Examples directory](http://code.google.com/p/gtm-oauth/source/browse/#svn/trunk/Examples).

## Adding the Controllers to Your Project ##

The project has targets for building a static library for iOS and a framework for Mac OS X. Alternatively, the source files and xibs may be dragged directly into your project file and compiled with your application.

Check out the "top-of-trunk" OAuth controller sources with a Mac terminal window and the command on the [source checkout page](http://code.google.com/p/gtm-oauth/source/checkout).
The source files required are:

|_iOS and Mac OS X_|_iOS_|_Mac OS X_|
|:-----------------|:----|:---------|
|GTMOAuthAuthentication.h/m<br>GTMOAuthSignIn.h/m<br>GTMHTTPFetcher.h/m<table><thead><th>GTMOAuthViewControllerTouch.h/m<br>GTMOAuthViewTouch.xib (optional)</th><th>GTMOAuthWindowController.h/m<br>GTMOAuthWindow.xib</th></thead><tbody></tbody></table>

These source files can be browsed in the project's <a href='http://code.google.com/p/gtm-oauth/source/browse/#svn/trunk/Source/'>source directory</a>.<br>
<br>
When linking against a static library build of the controller, specify the <a href='http://developer.apple.com/mac/library/qa/qa2006/qa1490.html'>-ObjC build option</a> for the application target's "Other Linker Flags".<br>
<br>
<h3>ARC Compatibility</h3>

When the controller source files are compiled directly into a project that has ARC enabled, then ARC must be disabled specifically for the controller source files.<br>
<br>
To disable ARC for source files in Xcode 4, select the project and the target in Xcode. Under the target "Build Phases" tab, expand the Compile Sources build phase, select the library source files, then press Enter to open an edit field, and type  <code>-fno-objc-arc</code>  as the compiler flag for those files.<br>
<br>
<h3>System Requirements</h3>

The Mac controller is compatible with Mac OS X 10.5 and later. The iOS controller is compatible with iPhone OS 3 and later.<br>
<br>
The OAuth controllers require linking to the system frameworks Security.framework and SystemConfiguration.framework.<br>
<br>
<br>
<h2>Signing in to Services</h2>

To sign in to a service, you should consult the service's API documentation to obtain a pre-assigned consumer key and secret string, and to find the required scope string for API operations.<br>
<br>
To use the OAuth 1 controllers with services, your application should create an authentication object with the appropriate keys, like this:<br>
<pre><code>- (GTMOAuthAuthentication *)myCustomAuth {<br>
  NSString *myConsumerKey = @"abcd";    // pre-registered with service<br>
  NSString *myConsumerSecret = @"efgh"; // pre-assigned by service<br>
<br>
  GTMOAuthAuthentication *auth;<br>
  auth = [[[GTMOAuthAuthentication alloc] initWithSignatureMethod:kGTMOAuthSignatureMethodHMAC_SHA1<br>
                                                      consumerKey:myConsumerKey<br>
                                                       privateKey:myConsumerSecret] autorelease];<br>
<br>
  // setting the service name lets us inspect the auth object later to know<br>
  // what service it is for<br>
  auth.serviceProvider = @"Custom Auth Service";<br>
<br>
  return auth;<br>
}<br>
</code></pre>

Displaying the sign-in view with a custom auth also requires providing the OAuth endpoints (URLs) and scope string to the controller, as shown here:<br>
<br>
<pre><code>- (void)signInToCustomService {<br>
<br>
  NSURL *requestURL = [NSURL URLWithString:@"http://example.com/oauth/request_token"];<br>
  NSURL *accessURL = [NSURL URLWithString:@"http://example.com/oauth/access_token"];<br>
  NSURL *authorizeURL = [NSURL URLWithString:@"http://example.com/oauth/authorize"];<br>
  NSString *scope = @"http://example.com/scope";<br>
<br>
  GTMOAuthAuthentication *auth = [self myCustomAuth];<br>
<br>
  // set the callback URL to which the site should redirect, and for which<br>
  // the OAuth controller should look to determine when sign-in has<br>
  // finished or been canceled<br>
  //<br>
  // This URL does not need to be for an actual web page<br>
  [auth setCallback:@"http://www.example.com/OAuthCallback"];<br>
<br>
  // Display the autentication view<br>
  GTMOAuthViewControllerTouch *viewController;<br>
  viewController = [[[GTMOAuthViewControllerTouch alloc] initWithScope:scope<br>
                                                              language:nil<br>
                                                       requestTokenURL:requestURL<br>
                                                     authorizeTokenURL:authorizeURL<br>
                                                        accessTokenURL:accessURL<br>
                                                        authentication:auth<br>
                                                        appServiceName:@"My App: Custom Service"<br>
                                                              delegate:self<br>
                                                      finishedSelector:@selector(viewController:finishedWithAuth:error:)] autorelease];<br>
<br>
  [[self navigationController] pushViewController:viewController<br>
                                         animated:YES];<br>
}<br>
</code></pre>

The application service name is used to save the token on the user’s keychain, and should identify both your application name and the service name(s). If appServiceName is nil, the token will not be saved, and the user will have to sign in again the next time the application is run.<br>
<br>
When the user signs in successfully or cancels signing in, the view or window controller will invoke your finishedSelector’s method:<br>
<pre><code>- (void)viewController:(GTMOAuthViewControllerTouch *)viewController<br>
      finishedWithAuth:(GTMOAuthAuthentication *)auth<br>
                 error:(NSError *)error {<br>
 if (error != nil) {<br>
   // Authentication failed<br>
 } else {<br>
   // Authentication succeeded<br>
 }<br>
}<br>
</code></pre>

If <code>[error code]</code> is kGTMOAuthErrorWindowClosed (-1000), then the user closed the sign-in view before completing authorization. Otherwise, any error reflects the server response in validating the user's access.<br>
<br>
The controllers also support Objective-C block completion handlers as alternatives to the delegate and finished selectors.<br>
<br>
<h3>Using the Authentication Tokens</h3>

If authentication succeeds, your application should retain the authentication object. It can be used directly to authorize NSMutableURLRequest objects:<br>
<br>
<pre><code>[auth authorizeRequest:myNSURLMutableRequest];<br>
</code></pre>

<h3>Retrieving Authorization from the Keychain</h3>

If your application saves the authorization to the keychain (by setting the controller's appServiceName), it can be retrieved the next time the application launches:<br>
<br>
<pre><code>- (void)awakeFromNib {<br>
  // Get the saved authentication, if any, from the keychain.<br>
  GTMOAuthAuthentication *auth = [self myCustomAuth];<br>
  if (auth) {<br>
    BOOL didAuth = [GTMOAuthViewControllerTouch authorizeFromKeychainForName:@"My App: Custom Service"<br>
                                                              authentication:auth];<br>
    // if the auth object contains an access token, didAuth is now true<br>
  }<br>
<br>
  // retain the authentication object, which holds the auth tokens<br>
  //<br>
  // we can determine later if the auth object contains an access token<br>
  // by calling its -canAuthorize method<br>
  [self setAuthentication:auth];<br>
}<br>
</code></pre>

If no authorization was saved, then “auth” will still be a valid authorization object but will be unable to authorize requests:<br>
<br>
<pre><code> BOOL isSignedIn = [auth canAuthorize]; // returns NO if auth cannot authorize requests<br>
</code></pre>

<h3>Signing Out</h3>

To completely discard the user’s authorization, use the view or window controller calls to remove the keychain entry:<br>
<pre><code> [GTMOAuthViewControllerTouch removeParamsFromKeychainForName:kAppServiceName];<br>
</code></pre>

Finally, release the authorization object.<br>
<br>
<h3>Debugging Tips</h3>

Getting OAuth 1 to work with a service provider can be challenging, especially with providers that implement extensions to the basic standard protocol. Here are a few times for getting GTMOAuth working.<br>
<br>
<h4>HTTP Logging</h4>

Turn on http logging to inspect the server requests and responses.  To enable http logging, add GTMHTTPFetcherLogging.h/m to your project, and call<br>
<pre><code>[GTMHTTPFetcher setLoggingEnabled:YES];<br>
</code></pre>
The log will be written to a folder on the desktop or in the device data directory.  Often, helpful server errors can be found in the responses in the log.<br>
<br>
More information about http logging is in the GTMHTTPFetcher documentation <a href='http://code.google.com/p/gtm-http-fetcher/wiki/GTMHTTPFetcherIntroduction#HTTP_Logging'>here</a>.<br>
<br>
<h4>Signature Base String</h4>

If the server signature check is failing, turn on signature diagnostics by specifying<br>
<pre><code>#define GTL_DEBUG_OAUTH_SIGNING 1<br>
</code></pre>
at the top of <code>GTMOAuthAuthentication.m</code>.  The console output will then list the components being signed and, more importantly, the base string being signed for a sign-in attempt.<br>
<br>
Compare the GTMOAuth base string to the base string shown in the provider's documentation.<br>
<br>
<h1>Design Notes</h1>

The library's classes are designed in three layers.<br>
<br>
<table><thead><th>Window/View Controller</th><th>user interface & application API</th></thead><tbody>
<tr><td>Sign-In               </td><td>networking (OAuth dance)        </td></tr>
<tr><td>Authentication        </td><td>data handling, request signing, and keychain</td></tr></tbody></table>

Classes are written to be independent of the layers above them.<br>
<br>
The window and view controllers are retained only during the user's sign-in interaction.<br>
<br>
The sign-in object is typically invisible to the client application.<br>
<br>
The authentication object must be retained by the client app to sign NSMutableURLRequests. It is also used to save authentication to and read authentication from the keychain.<br>
<br>
<h1>Questions and Comments</h1>

You can learn more about the OAuth protocol for desktop and mobile applications at <a href='http://code.google.com/apis/accounts/docs/OAuthForInstalledApps.html'>Google's documentation</a>.<br>
<br>
Additional documentation for the controllers is available in the header files.<br>
<br>
If you have any questions or comments about the library or this documentation, please join the <a href='http://groups.google.com/group/gtm-oauth'>discussion group</a>.