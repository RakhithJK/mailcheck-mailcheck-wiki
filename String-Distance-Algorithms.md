Mailcheck uses the sift3 string distance algorithm by default. Just pass your preferred string distance function into `distanceFunction` when using mailcheck.

We are always on the lookout for better alternatives, so send in your pull requests! Meanwhile, we have consolidated contributed alternatives below.

## Levenshtein Distance
Returns the strings' edit distance taking into account deletion, insertion, and substitution.

Contributed by [hpshelton](https://github.com/hpshelton).

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
    d[0][j] = j;
  }
              
  // Determine substring distances
  var cost = 0;
  for (var j = 1; j <= n; j++) {
    for (var i = 1; i <= m; i++) {
      cost = (s.charAt(i-1) == t.charAt(j-1)) ? 0 : 1;  // Subtract one to start at strings' index zero instead of index one
      d[i][j] = Math.min(d[i][j-1] + 1,                 // insertion
                         Math.min(d[i-1][j] + 1,        // deletion
                                  d[i-1][j-1] + cost)); // substitution                              
    }
  }
  
  // Return the strings' distance
  return d[m][n];
}
```
## 'Optimal' String-Alignment Distance
This is a variation of the Damerau-Levenshtein distance that returns the strings' edit distance taking into account deletion, insertion, substitution, and transposition, under the condition that no substring is edited more than once. 

For example, `optimalStringAlignmentDistance('ca', 'abc') == 3` because if the transposition 'ca' -> 'ac' is used, it is not possible to use the insertion 'ac' -> 'abc'. The shortest sequence of operations is 'ca' -> 'a' -> 'ab' -> 'abc'. 

Contributed by [hpshelton](https://github.com/hpshelton).

```javascript
function optimalStringAlignmentDistance(s, t) {
  // Determine the "optimal" string-alignment distance between s and t
  if (!s || !t) {
    return 99;
  }
  var m = s.length;
  var n = t.length;
  
  /* For all i and j, d[i][j] holds the string-alignment distance
   * between the first i characters of s and the first j characters of t.
   * Note that the array has (m+1)x(n+1) values.
   */
  var d = new Array();
  for (var i = 0; i <= m; i++) {
    d[i] = new Array();
    d[i][0] = i;
  }
  for (var j = 0; j <= n; j++) {
    d[0][j] = j;
  }
        
  // Determine substring distances
  var cost = 0;
  for (var j = 1; j <= n; j++) {
    for (var i = 1; i <= m; i++) {
      cost = (s.charAt(i-1) == t.charAt(j-1)) ? 0 : 1;   // Subtract one to start at strings' index zero instead of index one
      d[i][j] = Math.min(d[i][j-1] + 1,                  // insertion
                         Math.min(d[i-1][j] + 1,         // deletion
                                  d[i-1][j-1] + cost));  // substitution
                        
      if(i > 1 && j > 1 && s.charAt(i-1) == t.charAt(j-2) && s.charAt(i-2) == t.charAt(j-1)) {
        d[i][j] = Math.min(d[i][j], d[i-2][j-2] + cost); // transposition
      }
    }
  }
  
  // Return the strings' distance
  return d[m][n];
}
```
## Damerau-Levenshtein Distance
Returns the strings' edit distance taking into account deletion, insertion, substitution, and transposition, without the condition imposed by the 'optimal' string alignment distance algorithm above.

For example, `damerauLevenshteinDistance('ca', 'abc') == 2` because 'ca' -> 'ac' -> 'abc'. 

Contributed by [hpshelton](https://github.com/hpshelton).

```javascript
function damerauLevenshteinDistance(s, t) {
  // Determine the Damerau-Levenshtein distance between s and t
  if (!s || !t) {
    return 99;
  }
  var m = s.length;
  var n = t.length;      
  var charDictionary = new Object();
  
  /* For all i and j, d[i][j] holds the Damerau-Levenshtein distance
   * between the first i characters of s and the first j characters of t.
   * Note that the array has (m+1)x(n+1) values.
   */
  var d = new Array();
  for (var i = 0; i <= m; i++) {
    d[i] = new Array();
    d[i][0] = i;
  }
  for (var j = 0; j <= n; j++) {
    d[0][j] = j;
  }
  
  // Populate a dictionary with the alphabet of the two strings
  for (var i = 0; i < m; i++) {
    charDictionary[s.charAt(i)] = 0;
  }
  for (var j = 0; j < n; j++) {
    charDictionary[t.charAt(j)] = 0;
  }
  
  // Determine substring distances
  for (var i = 1; i <= m; i++) {
    var db = 0;
    for (var j = 1; j <= n; j++) {
      var i1 = charDictionary[t.charAt(j-1)];
      var j1 = db;
      var cost = 0;
      
      if (s.charAt(i-1) == t.charAt(j-1)) { // Subtract one to start at strings' index zero instead of index one
        db = j;
      } else {
        cost = 1;
      }
      d[i][j] = Math.min(d[i][j-1] + 1,                 // insertion
                         Math.min(d[i-1][j] + 1,        // deletion
                                  d[i-1][j-1] + cost)); // substitution
      if(i1 > 0 && j1 > 0) {
        d[i][j] = Math.min(d[i][j], d[i1-1][j1-1] + (i-i1-1) + (j-j1-1) + 1); //transposition
      }
    }
    charDictionary[s.charAt(i-1)] = i;
  }
        
  // Return the strings' distance
  return d[m][n];
}
```
## Performance
1000 randomly-generated email addresses were compared against the built-in domain list using the specified algorithm to determine an average comparison time. These average comparison times were then averaged over multiple experiments for some sort of semi-scientific-y conclusion. The following times were determined in Chrome on a MacBook Pro (3,1) with a 2.4 GHz Intel Core 2 Duo processor and 2 GB of RAM. Actual performance will vary by email addresses, domains, and client machine.

<table>
    <tr>
        <th>Algorithm</th>
        <th>Runtime</th>
    </tr>
    <tr>
        <td>Sift3 Distance</td>
        <td>0.03-0.04ms</td>
    </tr>
    <tr>
        <td>Levenshtein Distance</td>
        <td>0.10-0.20ms</td>
    </tr>
    <tr>
        <td>Optimal String Alignment Distance</td>
        <td>0.20-0.30ms</td>
    </tr>
    <tr>
        <td>Damerau-Levenshtein Distance</td>
        <td>0.40-0.50ms</td>
    </tr>
    <tr>
        <td>Qwerty Keyboard Distance</td>
        <td>1.00-1.55ms*</td>
    </tr>
</table>
*Caching affects the performance of this method, which results in very different runtimes depending on the input; the algorithm can perform as well as the Damerau-Levenshtein algorithm in some cases.