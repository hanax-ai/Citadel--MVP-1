### Deep Analysis of `5-crawl_site_recursively.py` (aka `5-crawl_recursive_internal_links.py`)

**File Purpose:** This script implements a recursive website crawler. Starting from a given set of seed URLs, it systematically discovers and crawls internal links on a website, level by level, up to a specified maximum depth. It leverages `crawl4ai`'s `arun_many` method for parallel crawling at each depth and employs a `MemoryAdaptiveDispatcher` for efficient resource management. Crucially, it includes logic for URL normalization and tracking visited URLs to prevent redundant crawling and infinite loops.

#### 1. Core Functionality & Workflow

1.  **Imports**:
    *   `asyncio`: For asynchronous programming.
    *   `urldefrag` from `urllib.parse`: A utility function from the Python standard library used to remove URL fragments (the part after a `#`). This is key for normalizing URLs.
    *   `AsyncWebCrawler`, `BrowserConfig`, `CrawlerRunConfig`, `CacheMode`, `MemoryAdaptiveDispatcher` from `crawl4ai`: These are advanced components from the `crawl4ai` library, consistent with previous examples using parallel and resource-aware crawling.

2.  **Recursive Batch Crawling Logic (`async def crawl_recursive_batch(start_urls, max_depth=3, max_concurrent=10)`)**:
    *   **Configuration Setup**:
        *   `browser_config = BrowserConfig(headless=True, verbose=False)`: Configures the crawler to use a headless browser and disables verbose logging from `crawl4ai`.
        *   `run_config = CrawlerRunConfig(cache_mode=CacheMode.BYPASS, stream=False)`: Sets the crawl run configuration to bypass caching (ensuring fresh data) and disable streaming (likely meaning content is fully loaded).
        *   `dispatcher = MemoryAdaptiveDispatcher(...)`: Initializes the dispatcher with parameters for memory threshold (70%), check interval (1 second), and maximum concurrent browser sessions (`max_concurrent`, defaulting to 10). This manages the parallelism of crawl tasks.
    *   **URL Tracking and Normalization**:
        *   `visited = set()`: A Python `set` is used to keep track of all unique, normalized URLs that have been processed or are queued for processing. This is vital for deduplication and preventing the crawler from getting stuck in infinite loops on sites with circular link structures.
        *   `def normalize_url(url): return urldefrag(url)[0]`: This helper function takes a URL string, uses `urldefrag` to split it into the main URL and the fragment identifier (e.g., `#section-name`), and returns only the main URL part. This ensures that `http://example.com/page` and `http://example.com/page#section` are treated as the same page for visitation tracking.
        *   `current_urls = set([normalize_url(u) for u in start_urls])`: The initial set of URLs to be crawled (at depth 1) is created from the `start_urls` list, with each URL being normalized.
    *   **Crawler Initialization**:
        *   `async with AsyncWebCrawler(config=browser_config) as crawler:`: The `AsyncWebCrawler` is instantiated using an asynchronous context manager, ensuring proper startup and shutdown of browser resources.
    *   **Depth-Limited Recursive Crawl Loop**:
        *   `for depth in range(max_depth):`: The main loop controls the recursion depth. It iterates from `0` to `max_depth - 1`.
        *   `print(f"
=== Crawling Depth {depth+1} ===")`: Logs the current depth level being processed.
        *   `urls_to_crawl = [normalize_url(url) for url in current_urls if normalize_url(url) not in visited]`: Before crawling, the `current_urls` list is filtered to include only those URLs that have not already been added to the `visited` set. This is a key deduplication step.
        *   `if not urls_to_crawl: break`: If the `urls_to_crawl` list is empty (meaning all discovered URLs at this level have already been visited or no new internal links were found in the previous depth), the recursion stops.
        *   **Parallel Crawling at Current Depth**:
            *   `results = await crawler.arun_many(urls=urls_to_crawl, config=run_config, dispatcher=dispatcher)`: The filtered list of `urls_to_crawl` for the current depth is processed in parallel using `crawler.arun_many()`. The `dispatcher` manages the concurrent execution.
        *   `next_level_urls = set()`: An empty set is initialized to store all new, unique, and unvisited internal URLs discovered during the current depth's crawl. These will form the `current_urls` for the next depth.
        *   **Processing Results and Discovering New Links**:
            *   The script iterates through each `result` object returned by `arun_many`.
            *   `norm_url = normalize_url(result.url)`: The URL from the result is normalized.
            *   `visited.add(norm_url)`: The normalized URL is added to the `visited` set, marking it as processed.
            *   `if result.success:`: If the crawl for this URL was successful:
                *   A success message is printed, including the URL and the length of the extracted Markdown content.
                *   `for link in result.links.get("internal", []):`: The script accesses `result.links`. This attribute is expected to be a dictionary where keys are link types (e.g., "internal", "external") and values are lists of link objects. It specifically retrieves "internal" links.
                *   `next_url = normalize_url(link["href"])`: Each discovered internal link's `href` attribute is normalized.
                *   `if next_url not in visited: next_level_urls.add(next_url)`: If the normalized internal link has not been visited, it's added to the `next_level_urls` set.
            *   `else:`: If `result.success` is false (crawl failed for this URL), an error message is printed.
        *   `current_urls = next_level_urls`: After processing all results for the current depth, the `next_level_urls` (containing newly discovered, unvisited internal links) becomes the `current_urls` for the next iteration of the depth loop.

3.  **Script Execution Block (`if __name__ == "__main__":`)**:
    *   `asyncio.run(crawl_recursive_batch(["https://ai.pydantic.dev/"], max_depth=3, max_concurrent=10))`:
        *   This line executes the `crawl_recursive_batch` function.
        *   It starts the crawl with a single seed URL: `"https://ai.pydantic.dev/"`.
        *   `max_depth=3`: The crawler will go up to 3 levels deep (seed URLs are depth 1).
        *   `max_concurrent=10`: Sets the maximum number of parallel browser sessions for the dispatcher.

#### 2. Key `crawl4ai` Features Demonstrated

*   **Iterative Recursive Crawling Pattern**: While `crawl4ai` doesn't offer a single "crawl recursively" command, this script masterfully constructs a recursive crawling mechanism by iteratively calling `arun_many` for each discovered level of URLs.
*   **Link Extraction (`result.links`)**: The script relies heavily on `crawl4ai`'s capability to parse HTML and extract links, categorizing them (e.g., as "internal"). The `result.links.get("internal", [])` access pattern is key to finding new pages to crawl within the same website.
*   **`arun_many` with `MemoryAdaptiveDispatcher`**: These features (also seen in script `3-crawl_sitemap_in_parallel.py`) are used here to efficiently crawl all URLs at a given depth in parallel, while managing system resources like memory.

#### 3. Algorithm and Logic Highlights

*   **Breadth-First-Like Traversal**: The crawler explores the website in a manner similar to a breadth-first search (BFS). It processes all pages at the current depth before moving to pages at the next depth.
*   **Effective Deduplication**: The combination of the `visited` set and URL normalization (using `urldefrag` to remove `#fragments`) is crucial for:
    *   **Efficiency**: Prevents re-crawling the same content multiple times.
    *   **Loop Prevention**: Avoids getting stuck in infinite loops if the website contains circular references between pages.
*   **Depth Control (`max_depth`)**: Provides a necessary constraint to limit the scope and duration of the crawl, especially important for large websites.
*   **Parallelism within Each Depth**: All URLs identified for a specific depth level are crawled concurrently, significantly speeding up the overall process compared to a purely sequential recursive approach.

#### 4. Error Handling and Robustness

*   **Handling of Individual URL Failures**: If `crawler.arun_many` encounters an error with a specific URL, the `result.success` flag for that URL will be false. The script logs this error and continues processing other URLs and subsequent depths.
*   **Resource Management**: The `MemoryAdaptiveDispatcher` plays a vital role in maintaining stability by preventing the crawler from consuming excessive memory, which is a common issue in large-scale parallel crawls.
*   **Infinite Loop Avoidance**: The `visited` set is the primary mechanism for preventing infinite loops.

#### 5. Dependencies

*   `asyncio` (Python standard library)
*   `urllib.parse` (Python standard library, specifically `urldefrag`)
*   `crawl4ai` (External library)

#### Implications for Architecture Document Enhancement:

*   **Crawling Layer (Advanced Strategy: Recursive Site Crawling)**:
    *   This script is an excellent blueprint for a comprehensive site crawling capability.
    *   The architecture document should detail this iterative, depth-limited recursive approach.
    *   Emphasize the critical roles of:
        *   URL normalization (e.g., using `urldefrag`) for accurate deduplication.
        *   The `visited` set for tracking processed URLs, preventing re-crawls and loops.
        *   `crawl4ai`'s link extraction feature (`result.links`) for discovering new internal pages.
        *   The use of `arun_many` and `MemoryAdaptiveDispatcher` at each depth to ensure scalable and resource-efficient parallel processing.
*   **URL Discovery and Scope Management**: This script directly addresses how to discover a broad set of relevant URLs from a starting point and how to manage the scope of the crawl (via `max_depth` and by focusing on internal links).
*   **Scalability and Performance Section**: The parallel execution within each depth level is a key contributor to the scalability of crawling entire websites or large sections thereof.
*   **Data Ingestion Pipeline**: This script represents a sophisticated method for the initial content acquisition phase, capable of gathering a wide array of documents from a target website based on its link structure.

This script is the most advanced among the examples, showcasing a robust and scalable solution for recursively crawling websites, a common requirement for comprehensive content gathering for RAG systems.
---
