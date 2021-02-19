# User Defined Functions (UDF) Library

To Contribute to the UDF Library
1. Grab UDF template located here [udf_template.md](udf_template.md)
2. Add your UDF to (this document) ~/UDF/README.md
3. Add your UDF to the [Table of Contents](#table-of-contents)
4. Issue a pull request

## Table of Contents
* [String Based UDF](#string-based-udf)
  * [substring](#substring) - Given a string of text, return a substring from index begin to index end
  * [str_regex_match](#str_regex_match) - Given a string determine if regex matches, return a boolean
  * [string_to_string](#string_to_string) -  string_to_string
  * [str_len](#str_len) - Given a string count char to get string size
  * [str_find](#str_find) - Given a string find a string in a string, and return where it was found (-1 if not found)
  * [regex_filter_set](#regex_filter_set) - Given a set and regular expression filter out strings and return the set
  * [regex_filter_list](#regex_filter_list) - Given a list and regular expression filter out strings and return the list
  * [double_to_string](#double_to_string) - Given a double output a string
  * [levenshtein_distance](#levenshtein_distance) - Levenshtein Distance for calculating string distance using Optimal string alignment distance.
 * [Integer/Float Based UDF](#integerfloat-based-udf)
   * [str_to_int](#str_to_int) - Given a string of numbers convert into an int
   * [float_to_int](#float_to_int) - Given a float convert it into an int
   * [echo_int](#echo_int) - Given an int echo an int
   * [rand_int](#rand_int) - Given a min and max int generate a random integer
* [Geo Based UDF](#geo-based-udf)
   * [getNearbyGridId](#getnearbygridid) - Given a distance in km and lat and lon return nearby
   * [geoDistance](#geodistance) - Given a a starting & ending lat and long calculate the distance

## String Based UDF
### substring
Given a string of text, return a substring from index begin to index end

| Variable | Description| Example |
| -------- | -------- | -------- |
| str    | input string of text     | The Apple is Red   |
| b    | index of string you would like to begin with     | index 4 = A     |
| e    |index of string you would like to end with    | index 8 = e|

**UDF Code**
```
inline string substring(string str, int b, int e) {
return str.substr (b,e);
}
```

**Example**
str = "The Apple is Red"
substring(s, 4, 8)

return = Apple

-----------

### str_regex_match
Given a string determine if regex matches, return a boolean


**UDF Code**
```
inline bool str_regex_match (string toCheck, string regEx) {
bool res = false;
// only performs the check when the input is valid
try {
  res = boost::regex_match(toCheck, boost::regex(regEx));
} catch (boost::regex_error err) {
  res = false;
}
return res;
}
```
**Example**
*Need to add*

-----------

### string_to_string
*Need to add*

**UDF Code**
```
  inline string to_string (double val) {
    char result[200];
    sprintf(result, "%g", val);
    return string(result);
  }
```
**Example**
*Need to add*

-----------

### str_len
Given a string count char to get string size

**UDF Code**
```
  inline int64_t str_len (string str) {
    return (int64_t) str.size();
  }
```
**Example**
*Need to add*

-----------

### str_find
Given a string find a string in a string, and return where it was found (-1 if not found)

**UDF Code**
```
 inline int64_t str_find (string str1, string str2) {
    return (int64_t) str2.find(str1);
  }
```
**Example**
*Need to add*

-----------

### regex_filter_set
Given a set and regular expression filter out strings and return the set

**UDF Code**
```
inline SetAccum <string> regex_filter_set (SetAccum <string>& inSet , string regEx) {
SetAccum <string> outSet;

for (auto& it :  inSet.data_) {
  if (str_regex_match(it, regEx)) {
    outSet += it;
  }
}
return outSet;
}
```
**Example**

```
CREATE QUERY tryFilterBag(string regex) FOR GRAPH MyGraph {
  /* Write query logic here */
	ListAccum <STRING> @outList ;
	SetAccum <STRING> @outSet;

	bv = {BagVertex.*};

	bv = select bvv from bv:bvv
	POST-ACCUM
	  bvv.@outList = regex_filter_list(bvv.myList, regex),
	  bvv.@outSet = regex_filter_set(bvv.mySet, regex);
      //HAVING bvv.@outList.size() > 0; // uncomment for direct filtration

	cv = select cvv from bv:cvv
	WHERE cvv.@outList.size() > 0 AND cvv.@outSet.size() > 0;

  PRINT bv, cv;
}
```

### regex_filter_list
Given a list and regular expression filter out strings and return the list

**UDF Code**
```
inline ListAccum <string> regex_filter_list (ListAccum <string>& inBag , string regEx) {
ListAccum <string> outBag;

for (int i=0; i < inBag.size(); i++){
    if (str_regex_match(inBag.get(i), regEx)) {
        outBag += inBag.get(i);
    }
}
return outBag;
}
```
**Example**
```
CREATE QUERY tryFilterBag(string regex) FOR GRAPH MyGraph {
  /* Write query logic here */
	ListAccum <STRING> @outList ;
	SetAccum <STRING> @outSet;

	bv = {BagVertex.*};

	bv = select bvv from bv:bvv
	POST-ACCUM
	  bvv.@outList = regex_filter_list(bvv.myList, regex),
	  bvv.@outSet = regex_filter_set(bvv.mySet, regex);
      //HAVING bvv.@outList.size() > 0; // uncomment for direct filtration

	cv = select cvv from bv:cvv
	WHERE cvv.@outList.size() > 0 AND cvv.@outSet.size() > 0;

  PRINT bv, cv;
}
```
### double_to_string
Given a double output a string

**UDF Code**
```
  inline string bigint_to_string (double val) {
    char result[200];
    sprintf(result, "%.0f", val);
    return string(result);
  }
```
**Example**
*Need to add*

### levenshtein_distance
Levenshtein Distance for calculating string distance using Optimal string alignment distance- https://en.wikipedia.org/wiki/Damerau%E2%80%93Levenshtein_distance

**UDF Code**
```
inline  int levenshtein_distance(string s1, string s2)
{
  int len_s1 = s1.length();
  int len_s2 = s2.length();
  int dist[len_s1+1][len_s2+1];

  int i;
  int j;
  int len_cost;

  for (i = 0;i <= len_s1;i++)
  {
      dist[i][0] = i;
  }
  for(j = 0; j<= len_s2; j++)
  {
      dist[0][j] = j;
  }
  for (i = 1;i <= len_s1;i++)
  {
      for(j = 1; j<= len_s2; j++)
      {
          if( s1[i-1] == s2[j-1] )
          {
              len_cost = 0;
          }
          else
          {
              len_cost = 1;
          }
          dist[i][j] = std::min(
          dist[i-1][j] + 1,                  // delete
          std::min(dist[i][j-1] + 1,         // insert
          dist[i-1][j-1] + len_cost)           // substitution
        );
       if( (i > 1) &&
       (j > 1) &&
       (s1[i-1] == s2[j-2]) &&
       (s1[i-2] == s2[j-1])
       )
       {
       dist[i][j] = std::min(
       dist[i][j],
        dist[i-2][j-2] + len_cost   // transposition
       );
       }
   }
}
return dist[len_s1][len_s2];
}
```
**Example**
```
CREATE QUERY levenshtein_distance(/* Parameters here */) FOR GRAPH *GRAPH_NAME* {
    String s1 = "jason";
    String s2 = "jasno";
    int dist;
    dist = levenshtein_distance(s1,s2);
    print dist;
}
```


-----------

## Integer/Float Based UDF

### echo_int
Given an int echo an int

**UDF Code**
```
  inline int64_t echo_int (int64_t echoThis) {
      return (int64_t) echoThis;
  }
```
**Example**
*Need to add*

-----------

### rand_int
Given a min and max int generate a random integer

**UDF Code**
```
inline int64_t rand_int (int minVal, int maxVal) {
  std::random_device rd;
  std::mt19937 e1(rd());
  std::uniform_int_distribution<int> dist(minVal, maxVal);
  return (int64_t) dist(e1);

}
```
**Example**
*Need to add*

-----------

### str_to_int
Given a string of numbers convert into an int

**UDF Code**
```
  inline int64_t str_to_int (string str) {
    return atoll(str.c_str());
  }
```
**Example**
*Need to add*

-----------

### float_to_int
Given a float convert it into an int

**UDF Code**
```
  inline int64_t float_to_int (float val) {
    return (int64_t) val;
  }
```
**Example**
*Need to add*

-----------

## Geo Based UDF

### getNearbyGridId
Given a distance in km and lat and lon return nearby

**UDF Code**
```
inline SetAccum<string> getNearbyGridId (double distKm, double lat, double lon) {

    string gridIdStr = map_lat_long_grid_id(lat, lon);
    uint64_t gridId = atoi(gridIdStr.c_str());

    int dia_long = gridNumLong (distKm, lat);
    int dia_lat = gridNumlat (distKm);

    int minus_dia_long = -1*dia_long;
    int minus_dia_lat  = -1*dia_lat;

    SetAccum<string> result;

    result += gridIdStr;

    int origin_lat = gridId/NUM_OF_COLS;
    int origin_lon = gridId%NUM_OF_COLS;

    for(int i = minus_dia_lat; i <= dia_lat; i++) {
      for(int j = minus_dia_long; j <= dia_long; j++) {
        int new_lat = origin_lat + i;
        int new_lon = origin_lon + j;

        // wrap around
        if (new_lat < 0) {
          new_lat = NUM_OF_ROWS + new_lat;
        } else if (new_lat > NUM_OF_ROWS) {
          new_lat = new_lat - NUM_OF_ROWS;
        }

        if (new_lon < 0) {
          new_lon = NUM_OF_COLS + new_lon;
        } else if (new_lon > NUM_OF_COLS) {
          new_lon = new_lon - NUM_OF_COLS;
        }

        int id = new_lon + NUM_OF_COLS * new_lat;

        result += std::to_string(id);
      }
    }
    return result;
  }
  ```
**Example**
*Need to add*

-----------

### geoDistance
Given a a starting & ending lat and long calculate the distance

**UDF Code**
```
  inline double geoDistance(double latitude_from, double longitude_from, double latitude_to, double longitude_to) {
    double phi_1 = deg2rad(latitude_from);
    double lambda_1 = deg2rad(longitude_from);
    double phi_2 = deg2rad(latitude_to);
    double lambda_2 = deg2rad(longitude_to);
    double u = sin(phi_2 - phi_1)/2.0;
    double v = sin(lambda_2 - lambda_1)/2.0;
    return 2.0 * earthRadiusKm * asin(sqrt(u * u + cos(phi_1) * cos(phi_2) * v * v));
  }
```
**Example**
*Need to add*
