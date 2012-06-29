---
layout: post
title: "Reload The Same Page Without Blinking On jQuery Mobile"
date: 2012-06-29 09:53
comments: true
categories: 
- jquery
- mobile
- javascript
- ios
---

When using jQuery Mobile, if you try to reload the current page from javascript using the usual tactics of assigning `window.location.href`, you might notice some unsightly artifacts and page blinking. This is particularly noticeable on iOS Mobile Safari (mainly on iPhone), because the reload also causes the address bar to slide down and back up.

Below is function I made to refresh the current page successfully with no visible artifacts: no blinking, no address bar showing, transitions:

```javascript
function refreshPage() {
  $.mobile.changePage(
    window.location.href,
    {
      allowSamePageTransition : true,
      transition              : 'none',
      showLoadMsg             : false,
      reloadPage              : true
    }
  );
}
```

_This was done on jQuery Mobile 1.1.0._
