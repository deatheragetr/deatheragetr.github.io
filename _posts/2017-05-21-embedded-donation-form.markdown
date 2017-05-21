---
layout: post
title:  "Creating an embedded donation form via funder.dog"
date:   2017-05-21 11:17:52 -0400
permalink: /create-an-embedded-donation-form-with-funderdog
categories: funder.dog donation form embedded
---
```html
  <iframe style="width: 100%; max-width: 449px; height: 609px; margin-top: 10px; margin-bottom: 10px; display: block; border-width: 0px;" id="funder-dog-donation-form-0001">
  </iframe>
  <script type="text/javascript">
    (function() {
      document.getElementById("funder-dog-donation-form-0001").src="http://donate.dev-funder.localhost:5000/cousteau-society?form-type=embedded";
     })()
  </script>
```

{% include embeddedDonationForm.html %}
