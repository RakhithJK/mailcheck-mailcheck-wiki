Mailcheck uses the sift3 string distance algorithm by default. We are always on the lookout for better alternatives, so send in your pull requests! Meanwhile, we have consolidated contributed alternatives below. You can pass them into the `Kicksend.mailcheck` method.

By [hpshelton](https://github.com/hpshelton):

```javascript
function levenshteinDistance(s, t) {
  // Determine the Levenshtein distance between s and t
  if (!s || !t) {
    return 99;
  }     
  
  var m = s.length;
  var n = t.length;
  
  /* For all i and j, d[i][j] holds the Levenshtein distance between
   * the first i characters of s and the first j characters of t.
   * Note that the array has (m+1)x(n+1) values.
   */
  var d = new Array();
  for (var i = 0; i <= m; i++) {
    d[i] = new Array();
    d[i][0] = i;
  }
  for (var j = 0; j <= n; j++) {
    d[0][j] = j
  }
        
  // Determine substring distances
  for (var j = 1; j <= n; j++) {
    for (var i = 1; i <= m; i++) {
      if (s[i-1] == t[j-1]) { // Subtract one to start at strings' index zero instead of index one
        d[i][j] = d[i-1][j-1];
      } else {
        d[i][j] = Math.min(d[i-1][j] + 1,            // deletion
                           Math.min(d[i][j-1] + 1,   // insertion
                                    d[i-1][j-1] + 1) // substitution
                          );
      }
    }
  }
  // Return the strings' distance
  return d[m][n];
}
```

