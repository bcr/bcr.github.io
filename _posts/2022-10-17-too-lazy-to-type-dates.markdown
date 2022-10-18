---
layout:     post
title:      Too Lazy to Type Dates
date:       2022-10-17T21:44:32-07:00
author:     Blake Ramsdell
categories: coding
tags:       python jekyll liquid
---
Jekyll wants the filename for a post to have the date. And wants a date in a
post. And I'm too lazy to work hard to make sure I got it right. So I made
Python do it.

Yes, there's probably some VSCode extension. So... lazy... Plus I got to learn
about dates and timezones in Python.

```python
#!/usr/bin/env python3

from datetime import datetime
import re
import sys

def dashimafy(title : str) -> str:
    title = title.lower()
    title = re.sub(r'c#', 'csharp', title)
    title = re.sub(r'\s+', r'_', title)
    title = re.sub(r'\W', '', title)
    title = re.sub(r'_', '-', title)
    return title

articledate = datetime.now()
articledate = articledate.astimezone()
title = sys.argv[1] if len(sys.argv) > 1 else input("Title, please: ")
author = "Blake Ramsdell"

filename = articledate.strftime('%Y-%m-%d') + '-' + dashimafy(title) + ".markdown"

contents = f"""---
layout:     post
title:      {title}
date:       {articledate.isoformat(timespec='seconds')}
author:     {author}
categories: 
tags: 
---
"""

with open(filename, "x") as output_file:
    print(contents, file=output_file)

print(f"New post {filename} created")
```
* `dashimafy` converts whatever lunacy you put in the title to the slugline for
  the filename.
* I used `isoformat` for the date because it's a standard dammit. Looks like
  Jekyll and Liquid don't puke on it.
