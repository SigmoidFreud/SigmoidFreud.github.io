---
layout: post
title: How Similar are Things According to Google?
category: Data Science

---

The following is just a code snippet that will tell give you back a continuous score between 0 and 1 as to how similar two things are according to google. The script is adapted from the Normalized Google Distance formula highlighted on wikipedia [here](http://en.wikipedia.org/wiki/Normalized_Google_distance).

{% highlight python %}
import google
import math
import sys
import json
import urllib

def googlesearch(searchfor):
    qu = urllib.urlencode({'q': searchfor})
    url = 'http://ajax.googleapis.com/ajax/services/search/web?v=1.0&%s' % qu
    response = urllib.urlopen(url)
    results = response.read()
    results = json.loads(results)
    data = results['responseData']
    return data

    args = sys.argv[1:]
    m = 50000000000
    if len(args) != 2:
        print "need two words as arguments"
        exit(0)
    fx = int(googlesearch(args[0])['cursor']['estimatedResultCount'])
    fy = int(googlesearch(args[1])['cursor']['estimatedResultCount'])
    fxy = int(googlesearch(args[0]+"+"+args[1])['cursor']['estimatedResultCount'])
    ngdnumerator = max(math.log10(fx),math.log10(fy))-math.log10(fxy)
    ngddenominator = math.log10(m)-min(math.log10(fx),math.log10(fy))
    ngd = ngdnumerator/ngddenominator
    print ngd
{% endhighlight %}
