## CTF Writeup: Tunnel

**Event:** Midnight Sun CTF **Category:** Web / Scripting **Points:** 129 **Solves:** 339

---

### Challenge Description

The challenge presents a simple web page with a binary string: `01101001 00100000 01101001 01101000 01100101 01100001 01110010 01110100 00100000 01100010 01101001 01101110 01100001 01110010 01111001` (which translates to "i heart binary").

Upon visiting the provided URL, the user is greeted with a page asking **"Which way did the flag go?"** and two hyperlinks containing unique `path` parameters.

### Initial Observation

Clicking the links leads to more pages, each offering two new links or a "DEAD END" message. This structure is a classic **directed graph traversal** problem. To find the flag, one must navigate through a "tunnel" (a maze of links) until the flag appears in the response body.

---

### Solution Strategy: Breadth-First Search (BFS)

Manual clicking is inefficient due to the depth and breadth of the "tunnel." A script is required to automate the process. **Breadth-First Search (BFS)** is the ideal algorithm here because:

1. It systematically explores all paths level by level.
    
2. It prevents getting stuck in infinite loops (if any) by tracking `visited` nodes.
    

#### The Script Logic

1. **Initialize:** Create a `queue` starting with the two initial paths and an empty `set` to track visited paths.
    
2. **Loop:**
    
    - Pop a path from the queue.
        
    - Send a GET request to the server with that path.
        
    - Check if the flag (starting with `midnight{`) is in the response.
        
    - If not, parse the response for new `path` values using regular expressions.
        
    - Add new, unvisited paths to the queue.
        
3. **Terminate:** Stop when the flag is found.
    

---

### Implementation (Python)

Python

```
import requests
import re
from collections import deque

BASE = "http://tunnel.play.ctf.se:22560/"
FLAG_PREFIX = "midnight{"
visited = set()
queue = deque(["z6qpcm5thexk", "41ebqfmu6onizs8"])

print("Starting the search...")

while queue:
    path = queue.popleft()
    if path in visited:
        continue
    visited.add(path)

    try:
        r = requests.get(BASE, params={"path": path}, timeout=10)
        text = r.text
    except Exception as e:
        print(f"Error fetching {path}: {e}")
        continue

    # Check for flag
    if FLAG_PREFIX in text:
        print(f"\n[!] FLAG FOUND at path={path}!")
        start = text.index(FLAG_PREFIX)
        end = text.index("}", start) + 1
        print(f"Flag: {text[start:end]}")
        break

    # Extract next paths using Regex
    new_paths = re.findall(r'\?path=([a-zA-Z0-9]+)', text)
    
    # Filter out already visited nodes and add to queue
    for p in new_paths:
        if p not in visited:
            queue.append(p)
    
    print(f"[{len(visited)}] Visited {path}... moving on.")
```

---

### Execution & Result

The script traverses the "tunnel," visiting various nodes. After **298 iterations**, the script reaches the correct path.

**Path Trace (Abbreviated):**

1. `z6qpcm5thexk` (Initial) -> Dead End
    
2. `41ebqfmu6onizs8` (Initial) -> Leads to more paths... ...
    
3. `uviswzk5qjl6h8g0` -> Leads to `156dfs3g0miv9h`
    
4. **`156dfs3g0miv9h`** -> **FLAG!**
    

**Flag:** `midnight{sh0uLD_h4v3_s3T_rob07s.txt}`

---

### Key Takeaways

- **Automation is key:** CTF web challenges involving large-scale link clicking are designed to be solved via scripting.
    
- **BFS vs DFS:** While Depth-First Search (DFS) would also work, BFS is often safer in web scraping to avoid getting trapped in deep, irrelevant branches.
