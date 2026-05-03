# Boosting Front-End Performance When an API Is Slow


---

## Table of Contents

1. [Overview](#overview)
2. [Why Slow APIs Affect Front-End Performance](#why-slow-apis-affect-front-end-performance)
3. [Main Strategies](#main-strategies)
   1. [Caching](#1-caching)
   2. [Lazy Loading](#2-lazy-loading)
   3. [Skeleton Screens and Loading Indicators](#3-skeleton-screens-and-loading-indicators)
   4. [Optimistic UI Updates](#4-optimistic-ui-updates)
   5. [Pagination and Infinite Scrolling](#5-pagination-and-infinite-scrolling)
   6. [Background Data Fetching](#6-background-data-fetching)
   7. [Debouncing and Throttling](#7-debouncing-and-throttling)
   8. [Batching API Calls](#8-batching-api-calls)
   9. [Using a CDN for Static Assets](#9-using-a-cdn-for-static-assets)
4. [Recommended Implementation Checklist](#recommended-implementation-checklist)
5. [Comparison Table](#comparison-table)
6. [Sample React Implementation](#sample-react-implementation)
7. [Common Mistakes to Avoid](#common-mistakes-to-avoid)
8. [Team Reference Summary](#team-reference-summary)

---

## Overview

When an API is slow, users may experience delays, blank screens, repeated loading states, or unresponsive interfaces. Even when the back-end cannot be fixed immediately, the front end can still improve the user experience by using smart performance techniques.

The article explains several front-end strategies that help reduce the impact of slow APIs:

- Caching
- Lazy loading
- Skeleton screens and spinners
- Optimistic UI updates
- Pagination or infinite scrolling
- Background fetching
- Debouncing and throttling
- Batching API calls
- CDN usage for static assets

The goal is not always to make the API faster directly. Instead, the front end should feel faster, reduce unnecessary requests, and keep the user informed while data is loading.

---

## Why Slow APIs Affect Front-End Performance

A slow API can create several front-end problems:

| Problem | Impact on User |
|---|---|
| Long waiting time | User may think the system is broken |
| Blank page while loading | Poor first impression |
| Repeated API calls | More network traffic and slower app |
| Large data loading | Browser may become slow |
| Delayed feedback after action | User may click multiple times |
| Failed requests without handling | User loses trust in the system |

A good front-end should handle slow APIs gracefully by:

1. Showing meaningful loading states.
2. Avoiding unnecessary API calls.
3. Loading only required data.
4. Reusing cached data when possible.
5. Giving quick visual feedback to users.
6. Handling errors clearly.

---

# Main Strategies

---

## 1. Caching

Caching means storing previously fetched data locally so the app does not need to request the same data again and again.

### Why Caching Helps

Caching is useful when:

- The same data is requested multiple times.
- The data does not change frequently.
- The user revisits the same page.
- The app needs to work faster after the first load.

### Common Front-End Caching Options

| Storage Option | Best For |
|---|---|
| `localStorage` | Small data that should remain after browser close |
| `sessionStorage` | Temporary data for one browser session |
| `IndexedDB` | Larger structured data |
| Service Worker Cache | Offline support and cached API/static responses |
| React Query / TanStack Query | API caching, background refetching, stale data handling |

### Basic `localStorage` Example

```js
// Save API response to localStorage
localStorage.setItem("userData", JSON.stringify(response.data));

// Read cached data
const cachedUserData = localStorage.getItem("userData");

if (cachedUserData) {
  const userData = JSON.parse(cachedUserData);
  console.log(userData);
}
```

### Safer Caching Example With Expiry Time

```js
const CACHE_KEY = "userData";
const CACHE_DURATION = 5 * 60 * 1000; // 5 minutes

function saveToCache(data) {
  const cacheItem = {
    data,
    timestamp: Date.now(),
  };

  localStorage.setItem(CACHE_KEY, JSON.stringify(cacheItem));
}

function getFromCache() {
  const cachedItem = localStorage.getItem(CACHE_KEY);

  if (!cachedItem) {
    return null;
  }

  const parsedItem = JSON.parse(cachedItem);
  const isExpired = Date.now() - parsedItem.timestamp > CACHE_DURATION;

  if (isExpired) {
    localStorage.removeItem(CACHE_KEY);
    return null;
  }

  return parsedItem.data;
}
```

### Example Usage

```js
async function fetchUserData() {
  const cachedData = getFromCache();

  if (cachedData) {
    return cachedData;
  }

  const response = await fetch("/api/user");
  const data = await response.json();

  saveToCache(data);

  return data;
}
```

### Important Notes

- Do not cache sensitive data such as passwords, tokens, or private financial information in `localStorage`.
- Always consider when cached data should expire.
- Use background refetching if the data needs to stay fresh.
- For complex apps, use tools like TanStack Query instead of manually managing cache.

---

## 2. Lazy Loading

Lazy loading means loading data only when it is needed instead of loading everything at once.

### Why Lazy Loading Helps

Lazy loading improves performance because:

- The first page loads faster.
- The browser handles less data at one time.
- The API receives smaller requests.
- Users only download data they actually need.

### Example: Fetch More Items by Page

```js
async function fetchMoreItems(page) {
  const response = await api.fetchItems(page);

  setItems((prevItems) => [
    ...prevItems,
    ...response.data,
  ]);
}
```

### React Example With “Load More” Button

```jsx
import { useEffect, useState } from "react";

function ItemList() {
  const [items, setItems] = useState([]);
  const [page, setPage] = useState(1);
  const [isLoading, setIsLoading] = useState(false);

  async function fetchItems(currentPage) {
    setIsLoading(true);

    try {
      const response = await fetch(`/api/items?page=${currentPage}`);
      const data = await response.json();

      setItems((prevItems) => [...prevItems, ...data.items]);
    } catch (error) {
      console.error("Failed to fetch items:", error);
    } finally {
      setIsLoading(false);
    }
  }

  useEffect(() => {
    fetchItems(page);
  }, [page]);

  return (
    <div>
      {items.map((item) => (
        <p key={item.id}>{item.name}</p>
      ))}

      <button
        onClick={() => setPage((prevPage) => prevPage + 1)}
        disabled={isLoading}
      >
        {isLoading ? "Loading..." : "Load More"}
      </button>
    </div>
  );
}

export default ItemList;
```

### When to Use Lazy Loading

Use lazy loading for:

- Product lists
- Search results
- Comments
- Notifications
- Image galleries
- Activity logs
- Tables with many records

---

## 3. Skeleton Screens and Loading Indicators

Skeleton screens and spinners are used while waiting for the API response.

### Skeleton Screen vs Spinner

| Loading UI | Best Use Case |
|---|---|
| Spinner | Short loading time or simple actions |
| Skeleton screen | Page/card/list layout where content is loading |
| Progress bar | Uploads, downloads, or multi-step processes |

### Why Skeleton Screens Help

Skeleton screens make the interface feel faster because users can see the structure of the page before real data appears.

Instead of showing a blank page, the system shows placeholder boxes that look like the final layout.

### Simple Skeleton Example

```jsx
function UserCardSkeleton() {
  return (
    <div className="user-card skeleton">
      <div className="skeleton-avatar"></div>
      <div className="skeleton-line"></div>
      <div className="skeleton-line short"></div>
    </div>
  );
}

export default UserCardSkeleton;
```

### CSS Example

```css
.skeleton {
  animation: pulse 1.5s infinite ease-in-out;
}

.skeleton-avatar {
  width: 48px;
  height: 48px;
  border-radius: 50%;
  background: #e0e0e0;
}

.skeleton-line {
  width: 100%;
  height: 12px;
  margin-top: 12px;
  border-radius: 4px;
  background: #e0e0e0;
}

.skeleton-line.short {
  width: 60%;
}

@keyframes pulse {
  0% {
    opacity: 1;
  }

  50% {
    opacity: 0.4;
  }

  100% {
    opacity: 1;
  }
}
```

### Example With Conditional Rendering

```jsx
function UserCard({ user, isLoading }) {
  if (isLoading) {
    return <UserCardSkeleton />;
  }

  return (
    <div className="user-card">
      <img src={user.avatar} alt={user.name} />
      <h3>{user.name}</h3>
      <p>{user.email}</p>
    </div>
  );
}
```

### Important Notes

- Skeleton screens should match the real layout.
- Do not show skeletons forever.
- Show a proper error message if the request fails.
- Use spinners for small actions, not full-page content.

---

## 4. Optimistic UI Updates

Optimistic UI means updating the interface immediately before waiting for the API response.

The front end assumes the API request will succeed. If the request fails, the front end rolls back the change.

### Why Optimistic UI Helps

Optimistic UI is useful because users get instant feedback.

For example:

- Liking a post
- Adding a comment
- Saving a task
- Marking a notification as read
- Updating a simple setting

### Basic Optimistic UI Example

```jsx
async function handleAddItem(newItem) {
  const previousItems = items;

  // Update UI immediately
  setItems((currentItems) => [...currentItems, newItem]);

  try {
    await api.addItem(newItem);
  } catch (error) {
    // Roll back if API fails
    setItems(previousItems);
    alert("Failed to add item. Please try again.");
  }
}
```

### More Complete Example

```jsx
function TodoList() {
  const [todos, setTodos] = useState([]);

  async function addTodo(title) {
    const temporaryTodo = {
      id: `temp-${Date.now()}`,
      title,
      completed: false,
    };

    const previousTodos = todos;

    setTodos((currentTodos) => [...currentTodos, temporaryTodo]);

    try {
      const response = await fetch("/api/todos", {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
        },
        body: JSON.stringify({ title }),
      });

      if (!response.ok) {
        throw new Error("Failed to save todo");
      }

      const savedTodo = await response.json();

      setTodos((currentTodos) =>
        currentTodos.map((todo) =>
          todo.id === temporaryTodo.id ? savedTodo : todo
        )
      );
    } catch (error) {
      setTodos(previousTodos);
      console.error(error);
    }
  }

  return (
    <div>
      {todos.map((todo) => (
        <p key={todo.id}>{todo.title}</p>
      ))}
    </div>
  );
}
```

### When Not to Use Optimistic UI

Avoid optimistic UI when:

- The action is high risk.
- The action involves payment.
- The action involves security permissions.
- The action may frequently fail.
- The user must see confirmed server data first.

---

## 5. Pagination and Infinite Scrolling

Pagination and infinite scrolling reduce the amount of data loaded at one time.

### Pagination

Pagination divides data into pages.

Example:

```txt
Page 1: Items 1 to 10
Page 2: Items 11 to 20
Page 3: Items 21 to 30
```

### Pagination Example

```jsx
import { useEffect, useState } from "react";

function PaginatedTable() {
  const [page, setPage] = useState(1);
  const [records, setRecords] = useState([]);
  const [isLoading, setIsLoading] = useState(false);

  useEffect(() => {
    async function fetchData() {
      setIsLoading(true);

      try {
        const response = await fetch(`/api/records?page=${page}&limit=10`);
        const data = await response.json();

        setRecords(data.records);
      } catch (error) {
        console.error("Failed to fetch records:", error);
      } finally {
        setIsLoading(false);
      }
    }

    fetchData();
  }, [page]);

  return (
    <div>
      {isLoading ? (
        <p>Loading records...</p>
      ) : (
        records.map((record) => (
          <p key={record.id}>{record.name}</p>
        ))
      )}

      <button
        onClick={() => setPage((currentPage) => currentPage - 1)}
        disabled={page === 1}
      >
        Previous
      </button>

      <button onClick={() => setPage((currentPage) => currentPage + 1)}>
        Next
      </button>
    </div>
  );
}

export default PaginatedTable;
```

### Infinite Scrolling

Infinite scrolling loads more data when the user scrolls near the bottom of the page.

### Infinite Scroll Example Using Intersection Observer

```jsx
import { useEffect, useRef, useState } from "react";

function InfiniteScrollList() {
  const [items, setItems] = useState([]);
  const [page, setPage] = useState(1);
  const loaderRef = useRef(null);

  async function fetchItems(pageNumber) {
    const response = await fetch(`/api/items?page=${pageNumber}`);
    const data = await response.json();

    setItems((currentItems) => [...currentItems, ...data.items]);
  }

  useEffect(() => {
    fetchItems(page);
  }, [page]);

  useEffect(() => {
    const observer = new IntersectionObserver((entries) => {
      const firstEntry = entries[0];

      if (firstEntry.isIntersecting) {
        setPage((currentPage) => currentPage + 1);
      }
    });

    const currentLoader = loaderRef.current;

    if (currentLoader) {
      observer.observe(currentLoader);
    }

    return () => {
      if (currentLoader) {
        observer.unobserve(currentLoader);
      }
    };
  }, []);

  return (
    <div>
      {items.map((item) => (
        <p key={item.id}>{item.name}</p>
      ))}

      <div ref={loaderRef}>Loading more...</div>
    </div>
  );
}

export default InfiniteScrollList;
```

### Pagination vs Infinite Scroll

| Approach | Best For |
|---|---|
| Pagination | Tables, admin dashboards, search results |
| Infinite scrolling | Feeds, galleries, social content |
| Load More button | Simple lists, mobile-friendly interfaces |

---

## 6. Background Data Fetching

Background fetching means loading data before the user needs it.

### Why Background Fetching Helps

It improves perceived performance because the data may already be available when the user opens the next screen.

### Example

```jsx
useEffect(() => {
  api.fetchNextBatch();
}, []);
```

### Better Example With Cache

```jsx
useEffect(() => {
  async function prefetchNextPage() {
    try {
      const response = await fetch("/api/items?page=2");
      const data = await response.json();

      sessionStorage.setItem("items-page-2", JSON.stringify(data));
    } catch (error) {
      console.error("Prefetch failed:", error);
    }
  }

  prefetchNextPage();
}, []);
```

### React Query / TanStack Query Example

```jsx
import { useQueryClient } from "@tanstack/react-query";

function ProductPage() {
  const queryClient = useQueryClient();

  function prefetchProductDetails(productId) {
    queryClient.prefetchQuery({
      queryKey: ["product", productId],
      queryFn: () => fetch(`/api/products/${productId}`).then((res) => res.json()),
    });
  }

  return (
    <button onMouseEnter={() => prefetchProductDetails(1)}>
      View Product
    </button>
  );
}
```

### When to Use Background Fetching

Use background fetching when:

- The next user action is predictable.
- The data is not too large.
- The API can handle the extra request.
- The prefetched data is likely to be used.

### Be Careful

Do not prefetch too much data because it can:

- Waste bandwidth.
- Increase API load.
- Slow down low-end devices.
- Affect users on mobile data.

---

## 7. Debouncing and Throttling

Debouncing and throttling reduce the number of API calls triggered by frequent user actions.

This is useful for:

- Search input
- Auto-complete
- Window resizing
- Scroll events
- Filtering
- Live validation

---

### Debouncing

Debouncing waits until the user stops typing before calling the API.

Example: Search runs only after the user stops typing for 500ms.

```js
function debounce(func, delay) {
  let debounceTimer;

  return function (...args) {
    clearTimeout(debounceTimer);

    debounceTimer = setTimeout(() => {
      func.apply(this, args);
    }, delay);
  };
}
```

### Debounced Search Example

```jsx
import { useMemo, useState } from "react";

function SearchBox() {
  const [query, setQuery] = useState("");

  async function searchProducts(searchTerm) {
    const response = await fetch(`/api/search?q=${searchTerm}`);
    const data = await response.json();

    console.log(data);
  }

  const debouncedSearch = useMemo(() => {
    return debounce(searchProducts, 500);
  }, []);

  function handleChange(event) {
    const value = event.target.value;

    setQuery(value);
    debouncedSearch(value);
  }

  return (
    <input
      value={query}
      onChange={handleChange}
      placeholder="Search products..."
    />
  );
}
```

---

### Throttling

Throttling limits how often a function can run.

Example: A scroll handler runs at most once every 500ms.

```js
function throttle(func, limit) {
  let inThrottle = false;

  return function (...args) {
    if (!inThrottle) {
      func.apply(this, args);
      inThrottle = true;

      setTimeout(() => {
        inThrottle = false;
      }, limit);
    }
  };
}
```

### Throttled Scroll Example

```js
const handleScroll = throttle(() => {
  console.log("Checking scroll position...");
}, 500);

window.addEventListener("scroll", handleScroll);
```

### Debounce vs Throttle

| Technique | Meaning | Best For |
|---|---|---|
| Debounce | Wait until action stops | Search input, form validation |
| Throttle | Limit action frequency | Scroll, resize, mouse movement |

---

## 8. Batching API Calls

Batching means combining multiple API requests into one request.

### Why Batching Helps

Batching helps reduce:

- Number of HTTP requests
- Network overhead
- Repeated authentication checks
- Waiting time from multiple round trips

### Example Without Batching

```js
const user = await fetch("/api/user/1");
const orders = await fetch("/api/user/1/orders");
const notifications = await fetch("/api/user/1/notifications");
```

This creates three separate API calls.

### Example With Batching

```js
const response = await fetch("/api/dashboard", {
  method: "POST",
  headers: {
    "Content-Type": "application/json",
  },
  body: JSON.stringify({
    userId: 1,
    include: ["profile", "orders", "notifications"],
  }),
});

const dashboardData = await response.json();
```

### Example API Response

```json
{
  "profile": {
    "id": 1,
    "name": "Aiman"
  },
  "orders": [
    {
      "id": 101,
      "item": "Keyboard"
    }
  ],
  "notifications": [
    {
      "id": 201,
      "message": "Your order has shipped"
    }
  ]
}
```

### When to Use Batching

Use batching when:

- Multiple related data requests are needed for one screen.
- The API supports batch endpoints.
- The requests are independent but needed together.
- You want to reduce network round trips.

### Important Notes

- Do not batch unrelated data unnecessarily.
- Avoid very large batch responses.
- Make sure one failed item does not break the whole response.
- Consider partial success responses.

---

## 9. Using a CDN for Static Assets

A CDN, or Content Delivery Network, helps deliver static files from servers closer to the user.

### Static Assets Include

- Images
- CSS files
- JavaScript bundles
- Fonts
- Icons
- Videos
- Documents

### Why CDN Helps

Even if the API is slow, static assets can still load quickly using a CDN.

This helps improve:

- Initial page load
- Image loading speed
- JavaScript delivery
- Global performance
- User experience for international users

### Example HTML With CDN Asset

```html
<img
  src="https://cdn.example.com/images/banner.webp"
  alt="Website banner"
/>
```

### Example Script From CDN

```html
<script src="https://cdn.example.com/libs/chart.min.js"></script>
```

### CDN Best Practices

- Use image compression.
- Use modern image formats like WebP or AVIF.
- Enable caching headers.
- Minify CSS and JavaScript.
- Use code splitting.
- Avoid loading unused libraries.
- Use lazy loading for images below the fold.

### Lazy Loading Image Example

```html
<img
  src="/images/product.webp"
  alt="Product image"
  loading="lazy"
/>
```

---

# Recommended Implementation Checklist

Use this checklist when improving a front-end application affected by slow APIs.

## User Interface

- [ ] Show skeleton screens for major page sections.
- [ ] Show loading states for buttons and forms.
- [ ] Disable repeated clicks while submitting.
- [ ] Display clear error messages when requests fail.
- [ ] Provide retry options where appropriate.

## API Request Optimization

- [ ] Cache data that does not change often.
- [ ] Use pagination for large datasets.
- [ ] Use lazy loading for below-the-fold content.
- [ ] Debounce search inputs.
- [ ] Throttle scroll or resize handlers.
- [ ] Batch related API calls when supported.

## Performance

- [ ] Compress images.
- [ ] Serve static assets through a CDN.
- [ ] Use code splitting.
- [ ] Remove unused JavaScript.
- [ ] Avoid unnecessary re-rendering.
- [ ] Prefetch likely next-page data carefully.

## Reliability

- [ ] Add timeout handling for slow requests.
- [ ] Add fallback UI for failed API calls.
- [ ] Roll back optimistic updates when API requests fail.
- [ ] Avoid caching sensitive data in browser storage.
- [ ] Set expiry time for cached data.

---

# Comparison Table

| Strategy | Purpose | Best Use Case | Risk |
|---|---|---|---|
| Caching | Reuse existing data | User profiles, settings, static lists | Stale data |
| Lazy loading | Load data only when needed | Large lists, images, comments | Delayed content |
| Skeleton screens | Improve perceived loading speed | Cards, dashboards, feeds | Misleading if loading takes too long |
| Optimistic UI | Give instant feedback | Likes, comments, simple updates | Rollback required |
| Pagination | Reduce data size per request | Tables, search results | More navigation clicks |
| Infinite scroll | Smooth continuous loading | Feeds, galleries | Harder to reach footer |
| Background fetching | Prepare data early | Predictable next actions | Extra API load |
| Debouncing | Reduce input-triggered requests | Search box | Slight delay |
| Throttling | Limit frequent event calls | Scroll, resize | Less real-time precision |
| Batching | Reduce number of requests | Dashboard data | Larger response size |
| CDN | Speed up static assets | Images, CSS, JS | Requires CDN setup |

---

# Sample React Implementation

The following example combines several techniques:

- Caching
- Loading state
- Error handling
- Pagination
- Debounced search

```jsx
import { useEffect, useMemo, useState } from "react";

function debounce(func, delay) {
  let timer;

  return function (...args) {
    clearTimeout(timer);

    timer = setTimeout(() => {
      func.apply(this, args);
    }, delay);
  };
}

const CACHE_DURATION = 5 * 60 * 1000;

function getCacheKey(searchTerm, page) {
  return `products:${searchTerm}:page:${page}`;
}

function saveToCache(key, data) {
  const cacheItem = {
    data,
    timestamp: Date.now(),
  };

  localStorage.setItem(key, JSON.stringify(cacheItem));
}

function readFromCache(key) {
  const cachedItem = localStorage.getItem(key);

  if (!cachedItem) {
    return null;
  }

  const parsedItem = JSON.parse(cachedItem);
  const isExpired = Date.now() - parsedItem.timestamp > CACHE_DURATION;

  if (isExpired) {
    localStorage.removeItem(key);
    return null;
  }

  return parsedItem.data;
}

function ProductList() {
  const [products, setProducts] = useState([]);
  const [searchTerm, setSearchTerm] = useState("");
  const [activeSearchTerm, setActiveSearchTerm] = useState("");
  const [page, setPage] = useState(1);
  const [isLoading, setIsLoading] = useState(false);
  const [errorMessage, setErrorMessage] = useState("");

  async function fetchProducts(term, currentPage) {
    const cacheKey = getCacheKey(term, currentPage);
    const cachedData = readFromCache(cacheKey);

    if (cachedData) {
      setProducts(cachedData.products);
      return;
    }

    setIsLoading(true);
    setErrorMessage("");

    try {
      const response = await fetch(
        `/api/products?q=${encodeURIComponent(term)}&page=${currentPage}`
      );

      if (!response.ok) {
        throw new Error("Failed to load products");
      }

      const data = await response.json();

      setProducts(data.products);
      saveToCache(cacheKey, data);
    } catch (error) {
      setErrorMessage("Unable to load products. Please try again.");
      console.error(error);
    } finally {
      setIsLoading(false);
    }
  }

  const debouncedSearch = useMemo(() => {
    return debounce((value) => {
      setActiveSearchTerm(value);
      setPage(1);
    }, 500);
  }, []);

  function handleSearchChange(event) {
    const value = event.target.value;

    setSearchTerm(value);
    debouncedSearch(value);
  }

  useEffect(() => {
    fetchProducts(activeSearchTerm, page);
  }, [activeSearchTerm, page]);

  return (
    <div>
      <input
        type="text"
        value={searchTerm}
        onChange={handleSearchChange}
        placeholder="Search products..."
      />

      {isLoading && <p>Loading products...</p>}

      {errorMessage && (
        <div>
          <p>{errorMessage}</p>
          <button onClick={() => fetchProducts(activeSearchTerm, page)}>
            Retry
          </button>
        </div>
      )}

      {!isLoading &&
        !errorMessage &&
        products.map((product) => (
          <div key={product.id}>
            <h3>{product.name}</h3>
            <p>{product.description}</p>
          </div>
        ))}

      <button
        onClick={() => setPage((currentPage) => currentPage - 1)}
        disabled={page === 1 || isLoading}
      >
        Previous
      </button>

      <button
        onClick={() => setPage((currentPage) => currentPage + 1)}
        disabled={isLoading}
      >
        Next
      </button>
    </div>
  );
}

export default ProductList;
```

---

# Common Mistakes to Avoid

## 1. Showing a Blank Page

Avoid showing nothing while waiting for data.

Better approach:

```jsx
if (isLoading) {
  return <SkeletonLayout />;
}
```

---

## 2. Calling the API on Every Keystroke

Bad example:

```jsx
function handleChange(event) {
  searchApi(event.target.value);
}
```

Better approach:

```jsx
const debouncedSearch = debounce(searchApi, 500);
```

---

## 3. Loading Too Much Data at Once

Bad example:

```js
fetch("/api/products");
```

Better approach:

```js
fetch("/api/products?page=1&limit=20");
```

---

## 4. No Error Handling

Bad example:

```js
const response = await fetch("/api/data");
const data = await response.json();
```

Better approach:

```js
try {
  const response = await fetch("/api/data");

  if (!response.ok) {
    throw new Error("Request failed");
  }

  const data = await response.json();
} catch (error) {
  console.error(error);
  showErrorMessage();
}
```

---

## 5. Caching Without Expiry

Bad example:

```js
localStorage.setItem("data", JSON.stringify(data));
```

Better approach:

```js
localStorage.setItem(
  "data",
  JSON.stringify({
    data,
    timestamp: Date.now(),
  })
);
```

---

# Team Reference Summary

## Main Idea

When an API is slow, front-end developers should not only wait for the back-end team to improve the API. The front end can still improve the experience using caching, loading states, pagination, debouncing, and other techniques.

## Best First Steps

1. Add loading states and skeleton screens.
2. Cache stable data.
3. Use pagination for large data.
4. Debounce search and filter inputs.
5. Add proper error handling.
6. Use optimistic UI only for safe low-risk actions.
7. Use CDN and image optimization for static assets.

## Recommended Priority

| Priority | Action |
|---|---|
| High | Add loading states and error handling |
| High | Add pagination or lazy loading |
| High | Debounce search API calls |
| Medium | Cache frequently reused data |
| Medium | Use optimistic UI for simple actions |
| Medium | Add background prefetching |
| Low to Medium | Add batching if API supports it |
| Low to Medium | Improve CDN/static asset delivery |

## Simple Rule

> Do not make the user wait on a blank screen.  
> Show progress, reuse data, reduce unnecessary requests, and load only what is needed.

---


## Notes From Heikal ~

This document is prepared as a practical reference guide based on the article. It expands the original ideas with clearer explanations, corrected code examples, and additional implementation notes suitable for front-end development work.
