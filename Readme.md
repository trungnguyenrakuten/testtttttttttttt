# DeepLink Android investigation

## 1.Links

### 1.1 Basic infor

- [Discourse topic](https://discourse.tech.rakuten-it.com/t/nothing-happens-to-the-app-integrated-with-id-sdk/5732/3)
- RakutenBrowser clientID : `rakuten_browser_mobile`
- RakutenBrowser package : `jp.co.rakuten.mobile.browser`
- Failure Chrome version : 99
- Successfull Chrome version : 84
- Android 10
- Error: **Nothing happend after selecting the Chrome option in the Launcher option.**

## 2. Conclusion

### 2.1 What cause the issue

As far as I know, There are 2 potential reasons

#### 2.1.1 Gesture time

Following this topic: [Chromium bug](https://bugs.chromium.org/p/chromium/issues/detail?id=1310013)
>The problem is XHRs and similar APIs like Fetch don't preserve the user gesture, so when a navigation to an app is performed as the result of an XHR or similar, if that XHR took more than 5 seconds the user gesture will have been lost and the app navigation will fail.

Solution: **NONE** - wait for response from Chromium team(last update 4days ago) 

#### 2.1.2 Navigation is blocked when it is not originaly from the Chrome instance.

When the response asks for redirecting to a deeplink, if there are more than 1 application that can handle the given link then the disambiguation dialog might appear.

Actutally, when user click the Chrome instance to solve the redirect action. It works. 
But it has been blocked by Chrome because:

```html
Navigation is blocked: jp.co.rakuten.idsdk.sample://localhost/oauth-callback?state=QFAi_br7rWdbdw97cT4IHQ&code=@St.key-for-various-tokens.mOJ23cakETfASaUc_CoiliLsxvyVwDEN.00vDJayvZRyLuocykwDKzW5Fms2jnoya1hLeRACSNy0hTapiZHj2uLPplJM_EOs4yiMfQmtnH7LeUXBPzBwh_rPvLCm-NF6UkJJRZ8m0ekeBhcmKwgoDzfFLlpwNZ7lIYzMLJqk9od5RNuHou5cgKANEGwnK2Vm-a4vAl9d2bJ-minKR63vzXAH8P2nXZzOnlie_O2MuLelYJusEPzdHcWUHSwEjviLF2QAGqJk8s9CDP5BwxXagjfbdaNMAUzXvGL3RYDHl5rBxbnjFRMXLonp-ZWEHt-TVWjOr1W78A0PuUgjQhq-xuxyI6Fm146Zy6SM5k1MozlBsJ47_prssYamheRXoAt28W-paKhvMz4645SwypJy_bsp15_fqEF42FyWg3HA5NkcfGHEJf5NggoeAEUOucb_mhDzKSbE7iEx0dFvl-ll6dWODKNI24q45kzpugAp8NhcV23vCdhsEQlih907oyMyYwtORBxHLTDnyA3s8Y5wWx583RYPQ0WjPWSuCbhTu3R2RYzTVR0t1AAxdSHhsWLjMyZUFEAoJLgmmgY9Spo6m67RVfi3EFRrYaWdOQugRDo-m8XLys6HF4D-r4GffdcK7loYImOGAzzYD9JGP26lS7_uZQ7woijQYvaR-yhog-22H0KcdH_munEUsPGUWr019UuQCGuwykhc9Q2JXbfYGPh9peSU45tGe6Nsy7fmlJ943A2Wf6Xtun_uxKzE548OoNWHD578RWCWKLi0KytddquGQETqO8y2A_ORPvxpBcTgkokulhNemoayD2QuxYRiQNmgilL2NXzuH4TqPVb8GtkMBleC27oAxOa6uJRriKwaYclAsA1vq5d881BMRfzKYNKqvNqu9mBwbLFQbqmT75d_dPA5QGgNCV7GBJV5K0d72Bqj2QSlqTU_TMbmhQPrW6ruYQPgOrZ-tqf0C2IBqe6vgNSsCcW4o70j6rBjLcoVpCFFLzQbLsegA1yPhX5pjaf222sCRltWAwa_V6WdpDarMfi-dVvP1w3txkAyJe5jNg2ytCLgstQZiBCrZN46NKE62KDDY8elOE4tW0YpfbOLguzftwNUdCnWWe9_On67zjoPMKEzum6Z_sr7C05dhtCqF82Nz16w8kEEDom3pTIwvGUAGKpQs1IiowqekIBmB0DgVj2CJ71zManFQe7TxCSNcD-kMBD94I5MW26gRpOpw3VFIIAtUf3gZCAaLjYRkZNtMS-XfzE4nOkcvqFYv2geuW38NnR5Z6GjeGPqlA0h3e8Lxg1JIOno9t04P1OC99kzJkMupef9B6NI4b4zcEn8Go1agvuOyXEs
```

- The OAuth session did not take in Chrome. It took place in Rakuten Browser so Chrome refuses to handle that.
- The Omni page do redirect by JS not the response from server. Not sure, need to verify the source of Omni.

Solution:

- **Recommend**: change to [Applinks](https://developer.android.com/training/app-links#android-app-links) for Android. We need to distingish Android AppLink and iOS deeplink. iOS system will handle the linking action so this will be not happen in iOS platform.
- Hot-fix(might help):
  - First, wait for Chrome 101, it will be resolved.
  - Second, some guy has told that wait more than 5 seconds will resolve the issue (Chromium will verify is that a person interact with chrome or not???. **need to be verified**)
  - Last one, If the Omni page is using JS to do the redirect action, we neeed to remove it and do it on the server side, return the response with the 302, response( need to check)
  
    if, the server just should return the response by using systax:

    ```js
        app.post('/redirect', (req, res) => {
            // The response code should be 301 or 302
            res.redirect("jp.co.rakuten.idsdk.sample://localhost/oauth-callback?state=QFAi_br7rWdbdw97cT4IHQ&code=@St.key-for-various-tokens.mOJ23cakETfASaUc_CoiliLsxvyVwDEN.00vDJayvZRyLuocykwDKzW5Fms2jnoya1hLeRACSNy0hTapiZHj2uLPplJM_EOs4yiMfQmtnH7LeUXBPzBwh_rPvLCm-NF6UkJJRZ8m0ekeBhcmKwgoDzfFLlpwNZ7lIYzMLJqk9od5RNuHou5cgKANEGwnK2Vm-a4vAl9d2bJ-minKR63vzXAH8P2nXZzOnlie_O2MuLelYJusEPzdHcWUHSwEjviLF2QAGqJk8s9CDP5BwxXagjfbdaNMAUzXvGL3RYDHl5rBxbnjFRMXLonp-ZWEHt-TVWjOr1W78A0PuUgjQhq-xuxyI6Fm146Zy6SM5k1MozlBsJ47_prssYamheRXoAt28W-paKhvMz4645SwypJy_bsp15_fqEF42FyWg3HA5NkcfGHEJf5NggoeAEUOucb_mhDzKSbE7iEx0dFvl-ll6dWODKNI24q45kzpugAp8NhcV23vCdhsEQlih907oyMyYwtORBxHLTDnyA3s8Y5wWx583RYPQ0WjPWSuCbhTu3R2RYzTVR0t1AAxdSHhsWLjMyZUFEAoJLgmmgY9Spo6m67RVfi3EFRrYaWdOQugRDo-m8XLys6HF4D-r4GffdcK7loYImOGAzzYD9JGP26lS7_uZQ7woijQYvaR-yhog-22H0KcdH_munEUsPGUWr019UuQCGuwykhc9Q2JXbfYGPh9peSU45tGe6Nsy7fmlJ943A2Wf6Xtun_uxKzE548OoNWHD578RWCWKLi0KytddquGQETqO8y2A_ORPvxpBcTgkokulhNemoayD2QuxYRiQNmgilL2NXzuH4TqPVb8GtkMBleC27oAxOa6uJRriKwaYclAsA1vq5d881BMRfzKYNKqvNqu9mBwbLFQbqmT75d_dPA5QGgNCV7GBJV5K0d72Bqj2QSlqTU_TMbmhQPrW6ruYQPgOrZ-tqf0C2IBqe6vgNSsCcW4o70j6rBjLcoVpCFFLzQbLsegA1yPhX5pjaf222sCRltWAwa_V6WdpDarMfi-dVvP1w3txkAyJe5jNg2ytCLgstQZiBCrZN46NKE62KDDY8elOE4tW0YpfbOLguzftwNUdCnWWe9_On67zjoPMKEzum6Z_sr7C05dhtCqF82Nz16w8kEEDom3pTIwvGUAGKpQs1IiowqekIBmB0DgVj2CJ71zManFQe7TxCSNcD-kMBD94I5MW26gRpOpw3VFIIAtUf3gZCAaLjYRkZNtMS-XfzE4nOkcvqFYv2geuW38NnR5Z6GjeGPqlA0h3e8Lxg1JIOno9t04P1OC99kzJkMupef9B6NI4b4zcEn8Go1agvuOyXEs")
        })
    ```

If the hot fix above can not solve the disambiguation dialog, we need to change to **Android Applink**.

### 2.2 Test project

Overview diagram

```plantuml

actor       User        as u
participant RakutenBrowser as rb
boundary    IDSDK    as id
participant CustomTab as ct
participant Omni as om
boundary    AppChooser    as ac
entity      RedirectURL      as r
participant Chrome as c
u -> rb : Ask for login
rb -> id : start oauth
id -> ct : start an Custom tab
ct -> om : load Omni content
om -> r : generate url and token
r -> ac : which app to use
alt user chooses Chrome
    ac -> c : Ask for open deeplink
    c -> c !! : "Navigation is blocked"\n(Nothing happend!)
else user choose RakutenBrowser
    ac -> ct : Ask for open the link
    ct -> id : close the Custom tab and pass the value
    id -> rb: success to get the token
end    


```

## 3. Android App Link - Recommendation solution

We have 3 types of deeplinks in Android platform

- DeepLink: we are here, using a Custom URIs
- WebLink
- Android Applink: Recommend to change to this.

```plantuml
start

- user click the link
if (the domain has defined the package) then (YES)
    :open the right package name;
else (NO) 
    : show intent chooser;
endif    

end

```

### Reset the state of Android App Links on a device

In our app, we do

> ./adb shell pm set-app-links --package jp.co.rakuten.mobile.browser 0 all

### Invoke the domain verification process

After you reset the state, you can perform the verification process
In our case, we do
> ./adb shell pm verify-app-links --re-verify jp.co.rakuten.mobile.browser

### [Check the verification result](https://developer.android.com/training/app-links/verify-site-associations#review-results)

In our case, we do:
> ./adb shell pm get-app-links jp.co.rakuten.mobile.browser

And for rakuten browser, we got:

```html
NOTHING
```

The result for success case should be:

```html

package.name.abc:
    ID: 01234567-89ab-cdef-0123-456789abcdef
    Signatures: [***]
    Domain verification state:
      example.com: verified
      sub.example.com: legacy_failure
      example.net: verified
      example.org: 1026
```

### 3.1 [Confirm the Digital Asset Links file](https://developer.android.com/training/app-links/verify-site-associations#test-dal-files)

Run the following cmd to query the verification state of the domains that your organization owns.
We do the test with the `https://stg.login.account.rakuten.com` by using: 

```url
https://digitalassetlinks.googleapis.com/v1/statements:list?source.web.site=https://stg.login.account.rakuten.com&relation=delegate_permission/common.handle_all_urls
```

And got the result:

```json
{
  "maxAge": "599.999999880s",
  "debugString": "********************* ERRORS *********************\n* Error: unavailable: Error fetching statements from https://stg.login.account.rakuten.com./.well-known/assetlinks.json (which is equivalent to 'https://stg.login.account.rakuten.com/.well-known/assetlinks.json'): URL_ERROR/3 [0] while fetching Web statements from https://stg.login.account.rakuten.com./.well-known/assetlinks.json (which is equivalent to 'https://stg.login.account.rakuten.com/.well-known/assetlinks.json') using download from the web (ID 1).\n********************* INFO MESSAGES *********************\n* Info: No statements were found that match your query\n",
  "errorCode": [
    "ERROR_CODE_FETCH_ERROR"
  ]
}
```

It seems there is not any file at the host of the `stg` env.

## 4. Debug support

Tool:

- [ADB](https://developer.android.com/studio/command-line/adb)
- [Chromimum github sample](https://github.com/kuoruan/Chromium-Android)

 Discussion thread

- [Android 6 show the disambiguation options](https://stackoverflow.com/questions/34359781/how-to-open-app-from-a-link-without-asking-user-to-decide-between-browser-or-app)
  - Verify applink (shoud be)
- [Ensure deeplink always opens the app](https://stackoverflow.com/questions/43441776/how-can-we-ensure-the-deep-link-will-always-open-up-our-own-native-app-instead-o)
  - Same as above link. Suggest to use `assetlinks.json`.
- [Another](https://stackoverflow.com/questions/61261073/how-can-android-deep-links-be-made-to-work-from-the-chrome-os-browser)
- [What Chromium do when the deeplink was clicked](https://stackoverflow.com/questions/36651441/android-app-https-deeplink-and-chrome-browser)
- [Solution for 'Navigation is blocked](https://stackoverflow.com/questions/41524087/navigation-is-blocked-when-redirecting-from-chrome-custom-tab-to-android-app)
- [Chromium topic talking about Oauth and this kind of error 'Navigation is blocked'](https://bugs.chromium.org/p/chromium/issues/detail?id=738724)