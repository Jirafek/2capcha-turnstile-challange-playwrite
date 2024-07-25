# 2capcha-turnstile-challange-playwrite
## Turnstile challange solver with 2 capcha (nodeJS)
#### userAgent changable only in firefox & chrome, for ios see this - [solution](https://gist.github.com/thorsten/148812e9cc4fb6a19215ce22afd4e5a8)

```javascript
await page.addInitScript(() => {
  const intervalId = setInterval(() => {
    if (window.turnstile) {
      clearInterval(intervalId);

      const originalRender = window.turnstile.render;

      window.turnstile.render = (a, b) => {
        const result = originalRender(a, b);
        const apiKey = 'API_KEY_HERE';

        const captchaParams = {
          method: "turnstile",
          key: apiKey,
          sitekey: b.sitekey,
          pageurl: window.location.href,
          data: b.cData,
          pagedata: b.chlPageData,
          action: b.action,
          json: 1
        };

        (async () => {
          try {
            const response = await fetch('https://2captcha.com/in.php', {
              method: 'POST',
              body: new URLSearchParams(captchaParams)
            });
            const responseData = await response.json();
            const requestId = responseData.request;

            let captchaSolution;
            while (!captchaSolution) {
              await new Promise(resolve => setTimeout(resolve, 5000));

              const resultResponse = await fetch(`https://2captcha.com/res.php?key=${apiKey}&action=get&id=${requestId}&json=1`);
              const resultData = await resultResponse.json();

              if (resultData.status === 1) {
                captchaSolution = resultData.request;
                const userAgent = resultData.useragent;

                if (navigator.__defineGetter__) {
                  navigator.__defineGetter__('userAgent', function () {
                    return userAgent;
                  });
                } else if (Object.defineProperty) {
                  Object.defineProperty(navigator, 'userAgent', {
                    get: function () {
                      return userAgent;
                    }
                  });
                }
              }
            }

            const input = document.querySelector('input[name="cf-turnstile-response"]');
            if (input) {
              input.value = captchaSolution;
            }

            window.tsCallback = b.callback;

            if (typeof window.tsCallback === 'function') {
              window.tsCallback();
            }
          } catch (error) {
            // Error exception
          }
        })();

        return result;
      };
    }
  }, 50);
});
