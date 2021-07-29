ESP: Encrypted Signals For Publishers
---

The Google Open Bidding Encrypted Signals for Publishers pilot allows OpenX to send encrypted signals directly from the page to our server-side bidder. For the pilot we will test with our 3rd party cookie. There are two main goals for the pilot:
1. Ensure the solution is technically sound.
2. Determine if ESP improves match rates and monetization compared to the current hosted cookie match table we use with Google OB.

In the future, if the pilot becomes generally available, we will expand the amount of signals we can support to improve match rate of other identifiers.

### How does it work

The Google Publisher Tag (GPT) now supports ESP through the registration of functions which return encrypted signals. Specifically functions are added to the `window.googletag.encryptedSignalSource` array. For OpenX, we will register a function under `window.googletag.encryptedSignalSource["openx"]` which returns our encrypted cookie identifier. GPT will then call our function to retrieve our signal after the page has loaded and whenever it is missing. The signal is cached in localStorage. In order to improve monetization, we will also cookie match with a few buyers at this point. On any subsequent ad request, GPT will append our signal to the server-side request. This signal will then be sent to OpenX through Open Bidding where we can decrypt and use it.

### Simple Implementation

OpenX has created a small ESP script which will do all of the necessary work to send the encrypted signals. The latest version can always be found at `https://oa.openxcdn.net/esp.js`. See the ChangeLog for links to specific versions. You must simply drop this script on your pages. The script can be loaded asynchronously and even loaded at the end of your page load to avoid any impact on user experience. In order for us to properly report on the change in match rate, you will have to drop the script on all pageviews of any domain you want to test on. Example:
```
<script async src="https://oa.openxcdn.net/esp.js"></script>
```

### Advanced Implementation

The ESP script exposes one function `window.ox_esp.getESPId` which will return a Promise resolving to the encrypted signal when available. You will not need to call this function as it will be done by GPT, however you can use it to test.

By default the ESP script will drop up to 5 cookie match pixels with our buyers to improve monetization on new cookies. You can edit the number of pixels which are dropped by appending an attribute `data-ox-plm` to the script with the maximum pixel limit. All pixels are dropped when the script is called by GPT, which will be after page load. Setting plm to 0 will disable buyer pixels.

By default the script will only log errors. If you wish to see debug messages in the console you can turn them on by setting the attribute `data-ox-debug=1`. Example:
```
<script async data-ox-plm=10 data-ox-debug=1 src="http://localhost:80/dist/esp.js"></script>
```

#### Verifying Implementation

To verify your implementation is working properly there are a few steps to take:
1. Force GPT to ask OpenX for an encrypted signal by deleting the current value from localStorage, likely found under `_GESPSK-openx`.
2. Verify a call to the OpenX endpoint `https://oajs.openx.net/esp` on the next page load.
3. Finally, if a value is returned by the endpoint (you will have to have 3rd party cookies enabled), then you can check the GPT call `https://securepubads.g.doubleclick.net/gampad/ads` for the query argument `a3p` which will contain encrypted signals.

If you provide a test page, OpenX will help verify the implementation.