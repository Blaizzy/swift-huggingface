# OAuth Module

The OAuth module provides secure authentication for Hugging Face services using OAuth 2.0 with PKCE (Proof Key for Code Exchange).

## Features

- **OAuth 2.0 with PKCE**: Secure authentication flow using industry-standard OAuth 2.0
- **Cross-Platform**: Works on iOS, macOS, watchOS, tvOS, and visionOS
- **Secure Token Storage**: Uses iOS/macOS Keychain for secure token persistence
- **Automatic Token Refresh**: Handles token refresh automatically
- **Comprehensive Scope Support**: Full support for all HuggingFace OAuth scopes

## Quick Start

### Basic Authentication

```swift
import OAuth

// Create an authentication manager
let authManager = try HuggingFaceAuthenticationManager(
    clientID: "your_client_id",
    redirectURL: URL(string: "yourapp://oauth/callback")!,
    scope: [.openid, .profile, .email],
    keychainService: "com.yourapp.huggingface",
    keychainAccount: "user_token"
)

// Sign in the user
try await authManager.signIn()

// Get a valid token for API calls
let token = try await authManager.getValidToken()
```

### Using Predefined Scope Sets

```swift
// Basic authentication (openid, profile, email)
let basicManager = try HuggingFaceAuthenticationManager(
    clientID: "your_client_id",
    redirectURL: URL(string: "yourapp://oauth/callback")!,
    scope: .basic,
    keychainService: "com.yourapp.huggingface",
    keychainAccount: "user_token"
)

// Read access to repositories
let readManager = try HuggingFaceAuthenticationManager(
    clientID: "your_client_id",
    redirectURL: URL(string: "yourapp://oauth/callback")!,
    scope: .readAccess,
    keychainService: "com.yourapp.huggingface",
    keychainAccount: "user_token"
)

// Full access (manage repos, inference API)
let fullManager = try HuggingFaceAuthenticationManager(
    clientID: "your_client_id",
    redirectURL: URL(string: "yourapp://oauth/callback")!,
    scope: .fullAccess,
    keychainService: "com.yourapp.huggingface",
    keychainAccount: "user_token"
)
```

### Custom Scope Configuration

```swift
// Custom scope combination
let customScopes: Set<HuggingFaceAuthenticationManager.Scope> = [
    .openid,
    .profile,
    .inferenceAPI,
    .writeDiscussions
]

let customManager = try HuggingFaceAuthenticationManager(
    clientID: "your_client_id",
    redirectURL: URL(string: "yourapp://oauth/callback")!,
    scope: customScopes,
    keychainService: "com.yourapp.huggingface",
    keychainAccount: "user_token"
)
```

### Advanced Configuration

```swift
// Custom OAuth client configuration
let config = OAuthClientConfiguration(
    baseURL: URL(string: "https://huggingface.co")!, // Default HuggingFace endpoint
    redirectURL: URL(string: "yourapp://oauth/callback")!,
    clientID: "your_client_id",
    scope: "openid profile email inference-api"
)

let oauthClient = OAuthClient(configuration: config)

// Custom token storage
let tokenStorage = HuggingFaceAuthenticationManager.TokenStorage(
    store: { token in
        // Custom storage implementation
        UserDefaults.standard.set(try JSONEncoder().encode(token), forKey: "oauth_token")
    },
    retrieve: {
        guard let data = UserDefaults.standard.data(forKey: "oauth_token") else { return nil }
        return try JSONDecoder().decode(OAuthToken.self, from: data)
    },
    delete: {
        UserDefaults.standard.removeObject(forKey: "oauth_token")
    }
)

let authManager = HuggingFaceAuthenticationManager(
    client: oauthClient,
    tokenStorage: tokenStorage
)
```

## Available Scopes

| Scope | Description |
|-------|-------------|
| `openid` | Get the ID token in addition to the access token |
| `profile` | Get the user's profile information (username, avatar, etc.) |
| `email` | Get the user's email address |
| `readBilling` | Know whether the user has a payment method set up |
| `readRepos` | Get read access to the user's personal repos |
| `writeRepos` | Get write/read access to the user's personal repos |
| `manageRepos` | Get full access to the user's personal repos |
| `inferenceAPI` | Get access to the Inference API |
| `writeDiscussions` | Open discussions and Pull Requests on behalf of the user |

## Predefined Scope Sets

| Set | Scopes Included |
|-----|----------------|
| `.basic` | `openid`, `profile`, `email` |
| `.readAccess` | `openid`, `profile`, `email`, `readRepos` |
| `.writeAccess` | `openid`, `profile`, `email`, `writeRepos` |
| `.fullAccess` | `openid`, `profile`, `email`, `manageRepos`, `inferenceAPI` |
| `.inferenceOnly` | `openid`, `inferenceAPI` |
| `.discussions` | `openid`, `profile`, `email`, `writeDiscussions` |

## Error Handling

```swift
do {
    try await authManager.signIn()
    let token = try await authManager.getValidToken()
    // Use token for API calls
} catch OAuthError.authenticationRequired {
    // User needs to authenticate
    print("Please sign in to continue")
} catch OAuthError.invalidConfiguration(let message) {
    // Configuration error
    print("Configuration error: \(message)")
} catch OAuthError.tokenExchangeFailed {
    // Token exchange failed
    print("Failed to exchange authorization code for token")
} catch {
    // Other errors
    print("Authentication error: \(error)")
}
```

## Sign Out

```swift
// Sign out the user
await authManager.signOut()

// Check authentication status
if authManager.isAuthenticated {
    // User is authenticated
    let token = try await authManager.getValidToken()
} else {
    // User needs to sign in
}
```

## Token Management

The authentication manager automatically handles:

- **Token Storage**: Securely stores tokens in the iOS/macOS Keychain
- **Token Refresh**: Automatically refreshes expired tokens
- **Token Validation**: Checks token expiration with a 5-minute buffer
- **Error Recovery**: Clears invalid tokens and requires re-authentication

## Security Features

- **PKCE Implementation**: Uses OAuth 2.0 PKCE for enhanced security
- **Secure Keychain Storage**: Tokens are stored securely in the system keychain
- **HTTPS Enforcement**: All OAuth endpoints must use HTTPS
- **Input Validation**: Comprehensive validation of all configuration parameters
- **Token Expiration**: Automatic token refresh with proper expiration handling

## Requirements

- iOS 16.0+ / macOS 14.0+ / watchOS 10.0+ / tvOS 17.0+ / visionOS 1.0+
- Swift 6.0+
- AuthenticationServices framework (for web authentication)
- CryptoKit framework (for PKCE implementation)

## Platform Support

The OAuth module works across all Apple platforms:

- **iOS**: Full support with `ASWebAuthenticationSession`
- **macOS**: Full support with `ASWebAuthenticationSession`
- **watchOS**: Limited support (no web authentication)
- **tvOS**: Limited support (no web authentication)
- **visionOS**: Full support with `ASWebAuthenticationSession`

## License

This module is part of the swift-huggingface package and follows the same license terms.
