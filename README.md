# Memorize
Creates a new function that, when invoked, caches the result of calling fn for a given argument set and returns the result.
Subsequent calls to the memoized fn with the same argument
set will not result in an additional call to fn;
instead, the cached result for that set of arguments will be returned.

take memorize of ramda as example:
```js
let count = 0;
const factorial = R.memoize(n => {
  count += 1;
  return R.product(R.range(1, n + 1));
});
factorial(5); //=> 120
factorial(5); //=> 120
factorial(5); //=> 120
count; //=> 1
```
## Cached Fetcher

the memorize talked before is based on pure function 
which give the same input always return the same output.
but fetch desn't satisfy. It causes side effects,
so generally, we need the expiry of the cache.

however, you shuold be ware of what you'r doing because
once the data stored at database  or anything could impact the
result changed and the cache is still alive, user will get a wrong data.

Here is the code:
```js
const cachedFetch = (url, options) => {
  let expiry = 5 * 60 // 5 min default
  if (typeof options === 'number') {
    expiry = options
    options = undefined
  } else if (typeof options === 'object') {
    // I hope you didn't set it to 0 seconds
    expiry = options.seconds || expiry
  }
  // Use the URL as the cache key to localStorage
  let cacheKey = url
  let cached = localStorage.getItem(cacheKey)
  let whenCached = localStorage.getItem(cacheKey + ':ts')
  if (cached !== null && whenCached !== null) {
    // it was in sessionStorage! Yay!
    // Even though 'whenCached' is a string, this operation
    // works because the minus sign converts the
    // string to an integer and it will work.
    let age = (Date.now() - whenCached) / 1000
    if (age < expiry) {
      let response = new Response(new Blob([cached]))
      return Promise.resolve(response)
    } else {
      // We need to clean up this old key
      localStorage.removeItem(cacheKey)
      localStorage.removeItem(cacheKey + ':ts')
    }
  }

  return fetch(url, options).then(response => {
    // let's only store in cache if the content-type is
    // JSON or something non-binary
    if (response.status === 200) {
      let ct = response.headers.get('Content-Type')
      if (ct && (ct.match(/application\/json/i) || ct.match(/text\//i))) {
        // There is a .json() instead of .text() but
        // we're going to store it in sessionStorage as
        // string anyway.
        // If we don't clone the response, it will be
        // consumed by the time it's returned. This
        // way we're being un-intrusive.
        response.clone().text().then(content => {
          localStorage.setItem(cacheKey, content)
          localStorage.setItem(cacheKey+':ts', Date.now())
        })
      }
    }
    return response
  })
}
```
## Reference
- [cache-fetched-ajax-requests](https://www.sitepoint.com/cache-fetched-ajax-requests/)
- [memoize](http://ramdajs.com/docs/#memoize)
