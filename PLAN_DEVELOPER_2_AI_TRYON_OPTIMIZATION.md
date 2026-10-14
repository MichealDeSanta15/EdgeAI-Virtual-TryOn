# 🧠 Developer 2 Plan: AI, Try-on Engine & Optimization
**Role**: You (Developer 2 - the complex one)  
**Focus**: Artificial intelligence, virtual try-on engine, VRAM optimization, and performance tuning  
**Collaboration Point**: Provides try-on generation service to Developer 1's frontend via standardized interface  
**Total Estimated Effort**: 16+ hours non-stop development  

## 🎯 Core Objectives
By the end of Day 1, you will have:
1. Stable Diffusion + ControlNet successfully installed and optimized for 8GB VRAM (RTX 5060)
2. A working try-on pipeline: person photo + garment image → realistic try-on result
3. Segmentation-based clothing masking functional
4. Basic inpainting generating recognizable outputs with proper garment alignment
5. Performance benchmarks showing <10 second try-on generation
6. Simple API for Developer 1 to integrate: `generate_tryon(person_path, garment_path) -> result_path`

By the end of Day 2, you will have:
- Advanced try-on quality improvements (lighting matching, texture preservation)
- Batch processing for multiple garments
- Enhanced post-processing (face enhancement, cloth simulation)
- Model warm-up/caching systems for sub-5 second generation
- AR mirror mode prototype exploration
- Production-ready AI service ready for integration

## 📅 Hour-by-Hour Breakdown

### **DAY 1: AI SETUP & CORE TRY-ON ENGINE** (Hours 1-16)

#### **Hours 1-4: ENVIRONMENT SETUP & BASE MODELS**
**Goal**: Working Stable Diffusion + ControlNet installation
- [ ] Verify RTX 5060 drivers and CUDA compatibility:
  - Check `nvidia-smi` shows CUDA 12.x compatible driver
  - Confirm 8GB VRAM available
- [ ] Install Python 3.10+ (if not present)
- [ ] Create clean virtual environment: `python -m venv ai_env && source ai_env/bin/activate`
- [ ] Install PyTorch with CUDA support:
  - `pip install torch torchvision --index-url https://download.pytorch.org/whl/cu121`
  - Verify: `python -c "import torch; print(torch.cuda.is_available())"`
- [ ] Install diffusers and transformers:
  - `pip install diffusers>=0.21.0 transformers>=4.30.0 accelerate>=0.20.0`
- [ ] Install xformers for memory optimization:
  - `pip install xformers==0.0.22` (or latest compatible)
- [ ] Install OpenCV and Pillow for image processing:
  - `pip install opencv-python>=4.8.0 Pillow>=10.0.0`
- [ ] Download base models (first time only - ~10GB total):
  - Stable Diffusion XL Base: `stabilityai/stable-diffusion-xl-base-1.0`
  - SDXL Refiner (optional but recommended): `stabilityai/stable-diffusion-xl-refiner-1.0`
  - ControlNet v1.1 models from lllyasviel:
    - `control_v11p_sd15_openpose` (pose)
    - `control_v11p_sd15_seg` (segmentation)
    - `control_v11p_sd15_canny` (edge)
    - `control_v11p_sd15_depth` (depth)
- [ ] Test basic text-to-image generation:
  - Generate: "a person wearing a red t-shirt, studio lighting"
  - Verify: Images produce without CUDA/OOM errors
- [ ] Deliverable by Hour 4: Working SDXL environment generating basic images

#### **Hours 5-8: CONTROLNET INTEGRATION & PREPROCESSING**
**Goal**: Functional ControlNet pipelines for pose and segmentation
- [ ] Implement ControlNet wrapper class:
  - Load multiple ControlNet models efficiently
  - Handle model switching/unloading to manage VRAM
  - Preprocessing pipelines for each control type
- [ ] Build pose estimation pipeline:
  - Option A: MediaPipe (fast, lightweight fallback)
    - Install: `pip install mediapipe`
    - Extract pose landmarks → draw on blank image → resize to 512x512
  - Option B: ControlNet OpenPose (higher quality)
    - Use `control_v11p_sd15_openpose` model
    - Preprocess: person image → pose detection → pose map
- [ ] Build segmentation pipeline:
  - Install: `pip install ultralytics` (for YOLOv8 segmentation) OR use DeepLab
  - Alternative: Use SAM (Segment Anything Model) for high-quality masks
  - Preprocess: person image → clothing mask (upper/lower body)
- [ ] Create unified conditioning system:
  - Accept: person image, garment image, control type (pose/seg/canny/depth)
  - Output: appropriate control maps for ControlNet
- [ ] Test: Generate image with pose control: "person wearing blue jeans" + pose map
- [ ] Deliverable by Hour 8: Functional pose and segmentation ControlNet pipelines

#### **Hours 9-12: TRY-ON ENGINE CORE & VRAM OPTIMIZATION**
**Goal**: Working try-on pipeline with memory optimization
- [ ] Implement segmentation-based clothing masking:
  - Input: person image
  - Output: binary mask where clothing should appear (upper body, lower body, full)
  - Techniques:
    - Option 1: Pre-trained segmentation model (DeepLabv3+)
    - Option 2: Fine-tuned segmentation on fashion datasets
    - Option 3: SAM (Segment Anything Model) with text prompts ("shirt", "pants")
  - Refine mask: morphological operations, smoothing, contour detection
- [ ] Develop garment preparation system:
  - Input: garment image from search results (various poses/backgrounds)
  - Output: standardized garment image (front view, transparent background)
  - Steps:
    1. Background removal (using rembg or similar)
    2. Pose normalization (ensure front-facing)
    3. Resize to standard dimensions (maintaining aspect ratio)
    4. Center on transparent canvas (512x512)
- [ ] Create alignment and warping module:
  - Input: person pose, garment pose, clothing mask
  - Output: warped garment image aligned to person's body
  - Techniques:
    - Thin-plate spline warping based on key points
    - Affine transformation for rough alignment
    - Mesh-based deformation for fine details
- [ ] Implement the core inpainting pipeline:
  1. Encode person image to latent space (VAE encoder)
  2. Create latent mask from clothing mask (1 where to inpaint, 0 to preserve)
  3. Prepare ControlNet conditioning (pose/seg maps)
  4. Run denoising loop with:
     - Text prompt: "[garment description] on a person, studio lighting, high quality"
     - Negative prompt: "deformed, bad anatomy, disfigured, poorly drawn"
     - ControlNet guidance: pose + segmentation maps
     - Latent masking: only denoise in clothing region
  5. Decode latent to image (VAE decoder)
  6. Post-process: blend with original person image at mask boundaries
- [ ] Apply VRAM optimization techniques:
  - xformers: Enable memory-efficient attention in UNet
  - torch.compile(): Compile UNet for faster iterations (PyTorch 2.0+)
  - CPU offloading: Move text encoder and VAE to CPU
  - Gradient checkpointing: Reduce activation memory
  - Model quantization: Experiment with FP16/INT8 where safe
  - Batch size = 1: Optimize for single image processing
- [ ] Test end-to-end try-on:
  - Person photo + t-shirt image → person wearing t-shirt
  - Person photo + dress image → person wearing dress
- [ ] Deliverable by Hour 12: Functional try-on engine with basic optimization

#### **Hours 13-16: PERFORMANCE TUNING & BASELINE BENCHMARKS**
**Goal**: Achieve target performance and prepare for integration
- [ ] Benchmark baseline performance:
  - Measure: VRAM usage (peak and steady state)
  - Measure: End-to-end latency (person + garment → try-on)
  - Measure: Image quality (visual inspection, subjective scoring)
- [ ] Identify and eliminate bottlenecks:
  - Profile with: `torch.profiler` or `nvprof`
  - Common culprits: VAE encoding/decoding, text encoder, ControlNet passes
- [ ] Apply targeted optimizations:
  - VAE: Use TAESD (Tiny AutoEncoder for Stable Diffusion) for faster decoding
  - Text encoder: Aggressive CPU offloading, quantization
  - ControlNet: Reduce inference steps where possible (distilled models)
  - UNet: Experiment with fewer denoising steps (LCM/Latta methods)
- [ ] Implement LCM LoRA for 2-4 step inference:
  - Download: `latent-consistency/lcm-lora-sdv1-5` (adapt for SDXL)
  - Or train simple LoRA on SDXL for consistency
  - Test: Quality vs. speed trade-off (aim for 3-4 sec generation)
- [ ] Enable model warm-up/caching:
  - Keep models loaded in VRAM between requests
  - Cache recent text embeddings
  - Pre-allocate common tensors
- [ ] Create simple API for Developer 1:
  - Function: `def generate_tryon(person_image_path, garment_image_path, output_path=None) -> str:`
  - Handles: loading, preprocessing, try-on generation, post-processing, saving
  - Returns: path to generated try-on image
  - Thread-safe: Can be called from QThread
- [ ] Test integration readiness:
  - Developer 1 can call: `result = generate_tryon("temp/person.jpg", "temp/garment.jpg")`
  - Output saved to: `temp/tryon_result.jpg`
  - Total time: <10 seconds on RTX 5060
- [ ] Deliverable by Hour 16: Optimized try-on engine with <10 sec latency and simple API

### **DAY 2: ADVANCED FEATURES & POLISH** (Hours 17-32)

#### **Hours 17-20: QUALITY ENHANCEMENTS & ADVANCED TECHNIQUES**
**Goal**: Improve try-on realism and robustness
- [ ] Enhance garment alignment:
  - Implement dense pose estimation for finer control
  - Use depth ControlNet for 3D-aware warping
  - Apply cloth simulation principles (basic drape simulation)
- [ ] Improve lighting and shading matching:
  - Extract lighting direction from person image
  - Adjust garment brightness/contrast to match scene
  - Add subtle shadows where garment contacts body
- [ ] Improve texture and detail preservation:
  - Use garbament image features to guide inpainting (Feature Fusion)
  - Implement latent blending techniques
  - Apply high-frequency detail preservation (wavelet methods)
- [ ] Address common failure modes:
  - Occlusion handling (when person blocks garment view)
  - Extreme poses (yoga, jumping, etc.)
  - Multi-layer clothing (jacket over shirt)
  - Transparent/sheer fabrics (special handling)
- [ ] Test with challenging cases:
  - Person in complex pose (arms crossed, turning)
  - Garment with intricate patterns (florals, stripes)
  - Person with accessories (hat, glasses, jewelry)
  - Low light or unusual lighting conditions
- [ ] Deliverable by Hour 20: Noticeably improved try-on quality on standard cases

#### **Hours 21-24: BATCH PROCESSING & EFFICIENCY**
**Goal**: Enable multiple rapid try-ons
- [ ] Implement batch try-on processing:
  - Input: 1 person image + N garment images
  - Output: N try-on results
  - Optimization: Share person encoding, reuse VAE/CLIP where possible
- [ ] Create intelligent batching system:
  - Group similar garments (same category) for shared preprocessing
  - Prioritize by user interaction (show first result ASAP)
  - Implement progressive refinement (show low-res first, enhance)
- [ ] Add caching layers:
  - Person encoding cache (last 5 person images)
  - Garment feature cache (last 20 garments)
  - LRU eviction to manage VRAM
- [ ] Implement asynchronous processing:
  - Non-blocking API: `generate_tryon_async(person, garment, callback)`
  - Progress reporting for batch operations
  - Cancellation support
- [ ] Test batch performance:
  - 1 person + 5 garments: target <25 seconds total (~5 sec/garment)
  - Memory usage stays <6GB VRAM
- [ ] Deliverable by Hour 24: Functional batch processing with caching

#### **Hours 25-28: ADVANCED POST-PROCESSING & AR EXPLORATION**
**Goal**: Polish outputs and explore future directions
- [ ] Implement face enhancement pipeline:
  - Detect face region in try-on result
  - Apply GFPGAN or CodeFormer for facial detail restoration
  - Blend gently to avoid "plastic" look
- [ ] Add cloth simulation and wrinkle generation:
  - Use basic physics equations for drape simulation
  - Generate normal maps for lighting enhancement
  - Apply subtle displacement based on pose and garment type
- [ ] Experiment with regional prompting:
  - Different prompts for different body parts (top vs bottom)
  - ControlNet weighting per region (seg map zones)
- [ ] Test alternative ControlNet combinations:
  - Pose + Depth for better 3D understanding
  - Canny + Seg for edge-aware segmentation
  - OpenPose alone for speed-critical scenarios
- [ ] Begin AR mirror mode exploration:
  - Research: real-time pose estimation + virtual garment overlay
  - Prototype: OpenCV + MediaPipe + simple garment overlay (no AI)
  - Benchmark: latency target <100ms for true real-time feel
  - Note: Full AI try-on AR requires significant optimization (future work)
- [ ] Deliverable by Hour 28: Enhanced outputs with face/garment improvements + AR prototype

#### **Hours 29-32: INTEGRATION, TESTING & FINAL PREP**
**Goal**: Seamless integration and presentation readiness
- [ ] Joint integration sprint with Developer 1 (Hours 29-30):
  - Receive test cases from Developer 1: person photos + search results
  - Run end-to-end benchmarks: total latency <15 seconds
  - Fix integration bugs: file paths, formats, threading issues
  - Jointly optimize: adjust based on real usage patterns
- [ ] Prepare performance benchmarks and documentation (Hours 31-32):
  - Create benchmark suite:
    * Latency: person + garment → try-on (target: <10 sec)
    * VRAM usage: peak and steady state (target: <6GB)
    * Quality: MOS (Mean Opinion Score) on test set
    * Throughput: garments per hour (target: >360 with batching)
  - Document API contract clearly:
    * Input formats, resolutions, color spaces
    * Output specifications
    * Error handling and return codes
    * Threading and concurrency notes
  - Write troubleshooting guide for common issues:
    * CUDA out of memory → reduce resolution, enable CPU offloading
    * Slow generation → check xformers, LCM LoRA, model warm-up
    * Poor quality → adjust prompts, try different ControlNet combos
  - Prepare final demo garments:
    * Curate 10 test garments across categories (t-shirt, shirt, dress, ethnic wear)
    * Create corresponding person photos in various poses/lighting
    * Generate expected outputs for validation
- [ ] Deliverable by Hour 32: Production-ready AI try-on service with benchmarks

## 🔧 Required Tools & Setup
**Before You Start**:
- [ ] RTX 5060 with 8GB VRAM (verify: `nvidia-smi`)
- [ ] CUDA 12.x compatible drivers
- [ ] Python 3.10+
- [ ] Create virtual environment: `python -m venv ai_env`
- [ ] Install core packages:
  ```
  pip install torch torchvision --index-url https://download.pytorch.org/whl/cu121
  pip install diffusers>=0.21.0 transformers>=4.30.0 accelerate>=0.20.0
  pip install xformers==0.0.22
  pip install opencv-python>=4.8.0 Pillow>=10.0.0
  pip install rembg>=2.0.0  # for background removal
  pip install scikit-image>=0.19.0  # for image processing
  pip install mediapipe>=0.10.0  # for pose estimation fallback
  ```
- [ ] Download models (first run only - ~12GB):
  - Make directory: `mkdir -p models/{sdxl,controlnet,vae,lora}`
  - Download to appropriate folders:
    - `models/sdxl/base/` → stabilityai/stable-diffusion-xl-base-1.0
    - `models/sdxl/refiner/` → stabilityai/stable-diffusion-xl-refiner-1.0 (optional)
    - `models/controlnet/openpose/` → lllyasviel/control_v11p_sd15_openpose
    - `models/controlnet/seg/` → lllyasviel/control_v11p_sd15_seg
    - `models/controlnet/canny/` → lllyasviel/control_v11p_sd15_canny
    - `models/controlnet/depth/` → lllyasviel/control_v11p_sd15_depth
    - `models/lora/lcm/` → latent-consistency/lcm-lora-sdv1-5 (adapt for SDXL)
- [ ] Create project structure:
  ```
  EdgeAI Virtual Try-On/
  ├── tryon_engine/
  │   ├── __init__.py
  │   ├── __main__.py
  │   ├── models/           # Model loading and management
  │   ├── preprocessing/    # Image preparation utilities
  │   ├── core/             # Try-on pipeline implementation
  │   ├── postprocessing/   # Result enhancement
  │   ├── api.py            # Developer-friendly interface
  │   └── benchmark.py      # Performance testing
  ├── models/               # Downloaded AI models
  ├── temp/                 # For image exchange with Developer 1
  ├── logs/                 # Generation logs and metrics
  └── benchmark_results/    # Performance test outputs
  ```

## 🤝 Integration Points with Developer 1
**File-Based Contract (Recommended for Simplicity & Reliability)**:
- **Input from Developer 1 → You**:
  - Person image: `temp/person_input.jpg` (RGB, 512x512 JPEG, quality 90+)
  - Garment image: `temp/garment_input.jpg` (RGB, 512x512 JPEG, quality 90+, front view)
- **Output from You → Developer 1**:
  - Try-on result: `temp/tryon_output.jpg` (RGB, 512x512 JPEG, quality 90+)
- **Alternative Direct API** (if preferred):
  - You expose in `tryon_engine/api.py`:
    ```python
    def generate_tryon(
        person_image_path: str,
        garment_image_path: str,
        output_path: str = None,
        num_inference_steps: int = 4,
        guidance_scale: float = 7.5,
        seed: int = None
    ) -> str:
        """
        Generate virtual try-on result.
        
        Args:
            person_image_path: Path to person photo (JPEG/PNG)
            garment_image_path: Path to garment image (JPEG/PNG)
            output_path: Where to save result (if None, returns temp path)
            num_inference_steps: Denoising steps (2-8, LCM LoRA enabled)
            guidance_scale: Classifier-free guidance strength
            seed: Random seed for reproducibility (None = random)
            
        Returns:
            Path to generated try-on image
        """
        # Implementation here
    ```
  - Developer 1 calls this function in a QThread

**Data Format Standards**:
- Images: JPEG, quality 90+, sRGB color space, 512x512 pixels
- Garment preparation: Front view, transparent background preferred
- Person image: Full body or upper body visible, well-lit
- Color space: sRGB (standard for web/images)
- Naming convention: Use timestamps to avoid conflicts: `person_1728901234.jpg`
- Cleanup: Developer 1 should delete temp files after successful display

## ⚠️ Common Challenges & Solutions
**CUDA Out of Memory (OOM)**:
- Problem: "CUDA out of memory. Tried to allocate X GiB"
- Solutions:
  - [ ] Enable xformers: `pipe.enable_xformers_memory_efficient_attention()`
  - [ ] CPU offload text encoder: `pipe.enable_model_cpu_offload()`
  - [ ] Reduce resolution: 512x512 → consider 384x384 for testing
  - [ ] Use torch.compile(): `pipe.unet = torch.compile(pipe.unet, mode="reduce-overhead")`
  - [ ] Gradient checkpointing: `pipe.enable_gradient_checkpointing()`
  - [ ] Sequential CPU offloading: `pipe.enable_sequential_cpu_offload()`

**Slow Generation Speed**:
- Problem: Taking >20 seconds per try-on
- Solutions:
  - [ ] Implement LCM LoRA: 2-4 steps instead of 20-30
  - [ ] Use TAESD decoder: 10x faster VAE decoding
  - [ ] Warm up models: Keep loaded between requests
  - [ ] Batch similar operations: Share CLIP/text encoder passes
  - [ ] Profile with: `torch.profiler.profile()` to find bottlenecks

**Poor Try-On Quality**:
- Problem: Garment looks deformed, misplaced, or unrealistic
- Solutions:
  - [ ] Improve garment preprocessing: better background removal, pose normalization
  - [ ] Enhance alignment: use pose key points for warping, not just bounding box
  - [ ] Adjust ControlNet weighting: try different combos (pose+seg vs pose+depth)
  - [ ] Refine masking: use more precise segmentation (SAM or DeepLabv3+)
  - [ ] Post-process: blend edges gently, preserve garment texture
  - [ ] Prompt engineering: be specific about garment type ("white cotton t-shirt")

**Integration Hiccups**:
- Problem: File not found, wrong format, permission errors
- Solutions:
  - [ ] Strictly enforce 512x512 RGB JPEG contract
  - [ ] Add image validation on both ends (dimensions, channels, color space)
  - [ ] Implement automatic conversion: resize, convert to RGB, save as JPEG
  - [ ] Use try/catch blocks with informative error messages
  - [ ] Test with known-good images before connecting systems

## 📈 Success Metrics (What "Done" Looks Like)
**By End of Day 1 (Hour 16)**:
- [ ] Stable Diffusion + ControlNet loaded successfully in <8GB VRAM
- [ ] Basic try-on pipeline functional: person + garment → recognizable output
- [ ] Segmentation mask creates reasonable clothing region
- [ ] End-to-end latency: <15 seconds per try-on
- [ ] VRAM usage: peak <7.5GB, steady state <6.5GB
- [ ] Simple API works: `generate_tryon("person.jpg", "garment.jpg") → "result.jpg"`
- [ ] Thread-safe: Can be called repeatedly from QThread
- [ ] Basic logging shows generation timing and VRAM usage

**By End of Day 2 (Hour 32)**:
- [ ] End-to-end latency: <10 seconds per try-on (target)
- [ ] VRAM usage: peak <6.5GB, steady state <5GB
- [ ] Batch processing: 5 garments in <35 seconds (~7 sec/garment)
- [ ] Try-on quality: garments appear naturally placed on person
- [ ] Edge artifacts: minimal visible seams or deformations
- [ ] Post-processing: faces look natural, colors blend well
- [ ] Robustness: handles 90% of test cases without visible failures
- [ ] Ready for integration: Developer 1 can connect search results to real try-ons

## 💡 Pro Tips for 2-Day Non-Stop Work
**Productivity Hacks for AI Development**:
- [ ] Use Jupyter notebooks for experimentation, then convert to scripts
- [ ] Keep a "known good" checkpoint: `git tag working-baseline-[date]`
- [ ] When stuck on optimization: try the simplest solution first (often works)
- [ ] Document everything: what you tried, what worked, what didn't (saves future time)
- [ ] Use tensorboardX or similar for logging experiments: `pip install tensorboard`
- [ ] Save intermediate outputs: debug images are invaluable for tuning
- [ ] Set up automatic model reloading: `watchdog` to detect changes and hot-reload
- [ ] Use `py-spy` or `perf` for performance profiling when needed

**Communication Protocol with Developer 1**:
- [ ] Start of day: Confirm API contract and file paths
- [ ] Every 3 hours: Share progress on quality/speed metrics
- [ ] Pre-integration: Exchange test images and expected outputs
- [ ] During integration: Work side-by-side on connecting systems
- [ ] Post-integration: Jointly run demo scenarios and fix issues together
- [ ] Emergency signal: If blocked >20 min on optimization, IM Developer 1 immediately

## 🎯 Your Critical Role
Remember: **You are building the core innovation** that makes this project magical. While Developer 1 creates the beautiful interface and search experience, *you* are creating the AI that actually lets users see themselves in clothes. When someone tries on a garment and thinks "Wow, that really looks like me on me" - that's your AI expertise shining through.

The optimization challenges you'll face (VRAM, speed, quality) are exactly what separates a demo from a usable product. Your work in squeezing Stable Diffusion + ControlNet into an 8GB RTX 5060 while maintaining quality is what makes this "Edge AI" rather than just another cloud-based try-on app.

You've got the skills to tackle this. One hour at a time, one optimization at a time, one beautiful try-on result at a time.
