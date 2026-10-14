# 🚀 Developer 1 Plan: Frontend, Camera & Search Systems
**Role**: Friend (Developer 1)  
**Focus**: User interface, camera integration, search functionality, and basic data management  
**Collaboration Point**: Integrates with Developer 2's AI try-on engine via standardized image inputs/outputs  
**Total Estimated Effort**: 16+ hours non-stop development  

## 🎯 Core Objectives
By the end of Day 1, you will have:
1. A functional camera interface showing phone/laptop video feed
2. A working search interface accepting natural language queries (e.g., "freshers dress under 5000")
3. A SQLite database populated with fashion product data
4. Basic integration allowing: camera view → capture photo → search query → view results
5. Foundation for Developer 2 to connect try-on engine

By the end of Day 2, you will have:
- Polished, professional UI/UX
- Multi-view try-on comparison system
- Advanced features (settings, guidance, logging)
- Virtual wardrobe management foundation
- Ready-for-demo application integrating with Developer 2's try-on outputs

## 📅 Hour-by-Hour Breakdown

### **DAY 1: FOUNDATION & SEARCH INFRASTRUCTURE** (Hours 1-16)

#### **Hours 1-4: PROJECT SETUP & CAMERA INTERFACE**
**Goal**: Working camera feed and basic UI structure
- [ ] Set up PyQt6 project structure (main.py, ui/ directories)
- [ ] Create MainWindow class with:
  - Left panel: Video display area (640x480 min)
  - Right panel: Search controls & results area
  - Status bar with progress indicator
- [ ] Implement OpenCV-based webcam simulator (use laptop webcam for testing)
- [ ] Add "Start/Stop Camera" button toggling webcam feed
- [ ] Display FPS counter and connection status
- [ ] Save captured frame as JPEG/PNG on button click
- [ ] Show captured image in secondary display area
- [ ] Test: Can start/stop camera, capture clear images, save to disk
- [ ] Deliverable by Hour 4: Functional camera simulator with capture

#### **Hours 5-8: SEARCH SYSTEM & DATABASE**
**Goal**: Working search with results display and data persistence
- [ ] Design SQLite database schema:
  - `products` table: id, name, description, price, category, image_path, brand, tags
  - `search_history` table: id, query, timestamp, results_count
  - `user_preferences` table: id, key, value
- [ ] Implement database helper class with CRUD operations
- [ ] Create sample dataset (20-30 fashion items):
  - Download/public domain fashion images
  - Create metadata (name, price ~₹500-5000, category, brand)
  - Store in `sample_dataset/` folder with thumbnails
- [ ] Populate database with sample data
- [ ] Build search interface:
  - Text input with placeholder: "e.g., freshers dress under 5000"
  - Price range slider (₹0-10000)
  - Search button (trigger on Enter/click)
  - Results list widget showing thumbnails + name/price
- [ ] Implement text-to-vector search (simple keyword matching for MVP)
- [ ] Display "X results found" status
- [ ] Test: Search returns relevant sample items, clicking shows details
- [ ] Deliverable by Hour 8: Searchable product database with UI

#### **Hours 9-12: WEB SCRAPING & DATA ENRICHMENT**
**Goal**: Automated data collection from fashion sites
- [ ] Research target sites: Myntra, Ajio, Amazon India (check robots.txt)
- [ ] Install Playwright and dependencies: `pip install playwright`
- [ ] Create scraper base class with:
  - Browser initialization (stealth mode)
  - Page navigation and waiting
  - Data extraction selectors (product name, price, image URL)
  - Error handling and retry logic
  - Rate limiting (1 request/second)
- [ ] Build site-specific scrapers:
  - Myntra: Extract from product cards
  - Ajio: Extract from grid items
  - Amazon: Use search results or product pages
- [ ] Implement image downloading and caching:
  - Save images to `cache/images/` with hash-based naming
  - Store local paths in database
  - Skip if already cached
- [ ] Create scraping controller with:
  - Start/stop/pause functions
  - Progress tracking (items processed/second)
  - Error logging and recovery
- [ ] Test: Scrape 10-20 items from one site, verify data quality
- [ ] Deliverable by Hour 12: Functional scraper adding real data to database

#### **Hours 13-16: INITIAL INTEGRATION & BASELINE**
**Goal**: Connect camera → capture → search pipeline
- [ ] Define integration contract with Developer 2:
  - Input: Person image (JPEG/PNG, 512x512 RGB)
  - Output: Try-on result image (same format)
  - File locations: `temp/person.jpg` ↔ `temp/tryon_result.jpg`
  - Simple API: Developer 2 provides `generate_tryon(person_path) -> result_path`
- [ ] Create integration test harness:
  - Simulate Developer 2's output with placeholder image
  - Test end-to-end: capture → search → (fake try-on) → display result
- [ ] Add "Generate Try-On" button (disabled until both images ready)
- [ ] Implement basic threading to prevent UI freezes:
  - QThread for camera capture
  - QThread for search operations
  - QThread for try-on generation (calling Developer 2's function)
- [ ] Add loading spinners and status messages
- [ ] Test full pipeline with mock try-on:
  1. Start camera
  2. Capture photo
  3. Enter search query
  4. Select result
  5. Click "Generate Try-On"
  6. Show processing indicator
  7. Display result (mock or real)
- [ ] Deliverable by Hour 16: Baseline working pipeline (camera → search → mock try-on)

### **DAY 2: POLISH, ADVANCED FEATURES & INTEGRATION** (Hours 17-32)

#### **Hours 17-20: UI/UX POLISH & ADVANCED FEATURES**
**Goal**: Professional, intuitive interface
- [ ] Apply consistent styling (QSS/theme):
  - Modern color palette (blues/greys with accent colors)
  - Rounded buttons, subtle shadows
  - Hover states and focus indicators
- [ ] Enhance search experience:
  - Autocomplete suggestions from popular queries/tags
  - Recent search history dropdown
  - Category/brand/filter chips
  - "Clear all" button
- [ ] Improve results display:
  - Grid view (3-4 columns) with hover zoom
  - List view option with detailed info
  - Sort by: price (low/high), relevance, newest
  - Wishlist/favorite button (heart icon)
- [ ] Add try-on workflow enhancements:
  - "Try This Outfit" button on each result
  - Side-by-side comparison: original vs. try-on
  - Before/after slider control
  - Multiple try-on comparison (2-3 garments simultaneously)
- [ ] Implement settings panel:
  - GPU optimization sliders (VRAM usage target)
  - Camera resolution/framerate settings
  - Output quality (try-on resolution: 512/768/1024)
  - Enable/disable face enhancement
- [ ] Add user guidance system:
  - Tooltips on hover for all controls
  - First-time user walkthrough tour
  - Contextual help button (?) in each section
  - Error messages with suggested fixes
- [ ] Test: Interface feels polished, responsive, and intuitive

#### **Hours 21-24: ADVANCED DATA & LOGGING**
**Goal**: Robust data handling and observability
- [ ] Enhance database:
  - Full-text search extension for better queries
  - Image similarity search foundation (placeholder for Dev 2)
  - User interaction logging (searches clicked, try-ons generated)
  - Export/import functionality for datasets
- [ ] Implement comprehensive logging:
  - Structured logging (JSON format) to file and console
  - Performance metrics: FPS, latency, VRAM usage
  - Error tracking with stack traces
  - User action analytics (anonymized)
- [ ] Add crash recovery:
  - Auto-save work-in-progress
  - Last session restoration
  - Corrupt database detection and repair
- [ ] Create diagnostics window:
  - Real-time GPU/memory usage graphs
  - Frame timing analysis
  - Search/query performance metrics
  - Try-on generation latency tracking
- [ ] Test: Logging captures all key events, diagnostics show useful data

#### **Hours 25-28: VIRTUAL WARDROBE & SOCIAL FEATURES (FOUNDATION)**
**Goal**: Prepare for future enhancements
- [ ] Design virtual wardrobe system:
  - `wardrobe` table: user_outfits (saved try-on combinations)
  - Outfit metadata: name, occasion, rating, notes
  - Garment tagging system (top/bottom/shoes/accessory)
  - Mix-and-match recommendation engine (placeholder)
- [ ] Implement basic wardrobe features:
  - "Save to Wardrobe" button on try-on results
  - Wardrobe gallery view (saved outfits)
  - Simple outfit builder (drag garments to person silhouette)
  - Outfit rating system (1-5 stars)
- [ ] Research privacy-first sharing options:
  - Local-only sharing (QR code for nearby devices)
  - Export as image/video (no personal data)
  - Anonymous analytics opt-in
- [ ] Create foundation for social features (stubs for future):
  - Sharing interface with privacy controls
  - Feedback collection mechanism
  - Waitlist/email capture foundation
- [ ] Test: Can save/load outfits, basic wardrobe navigation works

#### **Hours 29-32: INTEGRATION, TESTING & DEMO PREP**
**Goal**: Seamless integration and presentation readiness
- [ ] Joint integration sprint with Developer 2 (Hours 29-30):
  - Connect real try-on engine: replace mock with Developer 2's API
  - End-to-end testing: camera → search → real try-on → display
  - Performance benchmarking: target <15 sec total latency
  - Bug bashing: fix integration issues as they arise
- [ ] Polish and optimize (Hours 31-32):
  - Final UI/QA pass: fix alignment, spacing, typos
  - Performance tuning: reduce UI jank, optimize database queries
  - Prepare demo scenarios:
    * Scenario 1: "Freshers party under ₹2000"
    * Scenario 2: "Ethnic wear for wedding under ₹5000"
    * Scenario 3: "Professional attire for interview under ₹3000"
  - Create one-click demo launcher script
  - Write quick start guide for presentation
- [ ] Deliverable by Hour 32: Demo-ready integrated application

## 🔧 Required Tools & Setup
**Before You Start**:
- [ ] Install Python 3.10+
- [ ] Install PyQt6: `pip install PyQt6>=6.5.0`
- [ ] Install OpenCV: `pip install opencv-python>=4.8.0`
- [ ] Install numpy: `pip install numpy>=1.24.0`
- [ ] Install Pillow: `pip install Pillow>=10.0.0`
- [ ] Install Playwright: `pip install playwright && playwright install`
- [ ] Install SQLAlchemy: `pip install SQLAlchemy>=2.0.0`
- [ ] Install pytest: `pip install pytest`
- [ ] Create project structure:
  ```
  EdgeAI Virtual Try-On/
  ├── main.py
  ├── requirements.txt
  ├── ui/
  │   ├── __init__.py
  │   └── main_window.py
  ├── models/
  │   └── product.py
  ├── utils/
  │   ├── __init__.py
  │   └── database.py
  ├── sample_dataset/          # Will populate with fashion images
  ├── cache/                   # Image cache, thumbnails
  ├── temp/                    # For person/tryon image exchange
  ├── logs/                    # Application logs
  └── docs/                    # TECH_STACK.md, PROJECT_OVERVIEW.md
  ```

## 🤝 Integration Points with Developer 2
**File-Based Contract (Simple & Reliable)**:
- **Input from You → Developer 2**:
  - Person image: `temp/person_input.jpg` (RGB, 512x512 JPEG)
  - Garment image: `temp/garment_input.jpg` (RGB, 512x512 JPEG, from search result)
- **Output from Developer 2 → You**:
  - Try-on result: `temp/tryon_output.jpg` (RGB, 512x512 JPEG)
- **Alternative API Option** (if preferred):
  - Developer 2 exposes: `def generate_tryon(person_path, garment_path) -> result_path`
  - You call this function in a QThread when search result selected

**Data Format Standards**:
- Images: JPEG, quality 90+, sRGB color space
- Resolution: 512x512 (can scale to 768x768 if both agree)
- Naming: Use timestamps or UUIDs to avoid conflicts: `person_1728901234.jpg`
- Cleanup: Delete temp files after successful try-on generation

## ⚠️ Common Challenges & Solutions
**Camera Issues**:
- Problem: WebRTC/aiortc connection failures
  - Solution: Start with OpenCV/HTTP(MJPEG) fallback using IP Webcam/DroidCam apps
  - Solution: Implement connection retry with exponential backoff
  - Solution: Provide manual IP/port entry fallback

**Search Problems**:
- Problem: Scraper blocked by anti-bot measures
  - Solution: Implement realistic browser fingerprints (Playwright stealth)
  - Solution: Rotate user agents and add random delays
  - Solution: Cache aggressively to minimize requests
  - Solution: Fallback to pre-loaded dataset if scraping fails

**UI/Performance**:
- Problem: UI freezes during long operations
  - Solution: Always use QThreads for camera/search/try-on
  - Solution: Show progress indicators, not just spinners
  - Solution: Implement cancellation for long-running ops

**Integration Hiccups**:
- Problem: Image format/resolution mismatches
  - Solution: Strictly enforce 512x512 RGB JPEG contract
  - Solution: Add image validation/conversion pipeline on both sides
  - Solution: Test with known-good images before linking systems

## 📈 Success Metrics (What "Done" Looks Like)
**By End of Day 1 (Hour 16)**:
- [ ] Camera shows clear video feed (simulated or real)
- [ ] Capture button saves usable image to disk
- [ ] Search bar accepts queries and returns results from sample dataset
- [ ] Results list displays thumbnails with name/price
- [ ] "Generate Try-On" button enables when photo + search result selected
- [ ] Clicking try-on shows processing indicator + mock result
- [ ] Basic logging shows camera/search events

**By End of Day 2 (Hour 32)**:
- [ ] End-to-end latency: <15 seconds from capture to try-on display
- [ ] Try-on results are visually recognizable (garment discernible on person)
- [ ] UI remains responsive during all operations (no freezing)
- [ ] Application handles errors gracefully (no crashes)
- [ ] Demo scenarios run successfully from start to finish
- [ ] All logging/performance metrics collected and viewable
- [ ] Ready to present: "Point phone at yourself → search → see yourself in clothes"

## 💡 Pro Tips for 2-Day Non-Stop Work
**Productivity Hacks**:
- [ ] Use Pomodoro technique: 50 min work / 10 min break (stretch, hydrate, eyes rest)
- [ ] Keep water and snacks at desk to minimize context switching
- [ ] Use dual monitors if possible: code + reference/docs
- [ ] Commit code frequently: `git add . && git commit -m "WIP: [feature]"`
- [ ] Use issue tracking: GitHub issues or simple TODO.txt for blockers
- [ ] Pair-program integration hours: Work together on connecting systems
- [ ] Celebrate small wins: "Camera working!", "Search returns results!", "First try-on generated!"

**Communication Protocol**:
- [ ] Start of day (Hour 0): 15-min sync on plans and blockers
- [ ] Every 4 hours: Quick status check (what done, what next, blockers)
- [ ] End of day (Hour 16): 30-min retrospective + Day 2 planning
- [ ] Use shared notes doc for: API contracts, file paths, test data, known issues
- [ ] Emergency signal: If blocked >30 min, IM the other developer immediately

**You've Got This!** Remember: Your role delivers the user-facing magic that makes the AI try-on feel effortless and delightful. The search and camera systems are what users interact with first—make them polished, intuitive, and reliable. When someone sees themselves in clothes through your interface and thinks "Wow, that actually looks like me!", that's your success shining through.
