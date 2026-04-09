<template>
  <main ref="main" class="page-shell" :class="{ 'menu-open': isMobileMenuOpen }">
    <header ref="siteNav" class="site-nav">
      <button
        class="menu-toggle"
        type="button"
        :aria-expanded="isMobileMenuOpen"
        aria-controls="mobile-menu"
        aria-label="Open navigation"
        @click="toggleMobileMenu"
      >
        <span></span>
        <span></span>
        <span></span>
      </button>
      <button class="brand-signature" type="button" @click="navigateToSection('hero')">
        <span class="brand-dot"></span>
        Atelier Drive
      </button>
      <nav class="nav-links" aria-label="Primary">
        <button type="button" @click="navigateToSection('hero')">About</button>
        <button type="button" @click="navigateToSection('stories-intro')">Stories</button>
        <button type="button" @click="navigateToSection(stories[0].id)">Cars</button>
        <button type="button" @click="navigateToSection('reserve')">Contact</button>
      </nav>
      <div class="nav-actions">
        <button class="theme-toggle" type="button" :aria-pressed="isDarkMode" :aria-label="themeButtonLabel" @click="toggleTheme">
          <span class="theme-toggle-track" aria-hidden="true">
            <span class="theme-toggle-thumb"></span>
          </span>
          <span>{{ isDarkMode ? 'Light' : 'Dark' }}</span>
        </button>
        <button class="nav-cta" type="button" @click="navigateToSection('reserve')">Schedule Visit</button>
      </div>
    </header>

    <button
      class="menu-backdrop"
      type="button"
      :class="{ open: isMobileMenuOpen }"
      aria-label="Close navigation"
      @click="closeMobileMenu"
    ></button>

    <aside id="mobile-menu" class="mobile-menu" :class="{ open: isMobileMenuOpen }" aria-label="Mobile navigation">
      <p class="mobile-menu-label">Menu</p>
      <div class="mobile-menu-links">
        <button type="button" @click="navigateToSection('hero')">About Atelier</button>
        <button type="button" @click="navigateToSection('stories-intro')">Featured Stories</button>
        <button type="button" @click="navigateToSection(stories[0].id)">Car Gallery</button>
        <button type="button" @click="navigateToSection('reserve')">Contact Studio</button>
      </div>
    </aside>

    <div class="page-content">
      <section id="hero" ref="heroCard" class="hero-card tilt-surface" @mousemove="onTiltMove" @mouseleave="onTiltLeave">
        <div class="hero-copy">
          <Transition name="swap-fade" mode="out-in">
            <div :key="activeBrand.id" class="hero-copy-panel">
              <p class="eyebrow">{{ activeBrand.eyebrow }}</p>
              <h1 class="hero-title">{{ activeBrand.heroTitle }}</h1>
              <p class="hero-description">{{ activeBrand.heroDescription }}</p>
              <div class="hero-actions">
                <button class="primary-btn" type="button" @click="scrollToSection(activeBrand.storyId)">Explore {{ activeBrand.label }}</button>
                <button class="ghost-btn" type="button" @click="openBrandGallery(activeBrand.id)">View Gallery</button>
              </div>
              <div class="hero-facts">
                <div v-for="fact in activeBrand.facts" :key="fact.label" class="fact-card">
                  <strong>{{ fact.value }}</strong>
                  <span>{{ fact.label }}</span>
                </div>
              </div>
            </div>
          </Transition>
        </div>

        <div class="hero-visual">
          <div class="hero-ring ring-one"></div>
          <div class="hero-ring ring-two"></div>
          <Transition name="swap-fade" mode="out-in">
            <div :key="`${activeBrand.id}-visual`" class="hero-visual-panel">
              <div class="hero-glass-card hero-glass-top">{{ activeBrand.topTag }}</div>
              <div class="hero-glass-card hero-glass-bottom">{{ activeBrand.bottomTag }}</div>
              <img
                class="hero-float hero-float-back interactive-media"
                :class="{ 'vertical-orient-detail': activeBrand.isVertical }"
                :src="activeBrand.heroDetailImage"
                :alt="`${activeBrand.label} detail`"
                loading="eager"
                decoding="async"
                fetchpriority="low"
                role="button"
                tabindex="0"
                :aria-label="`View ${activeBrand.label} detail image`"
                @click="openImageViewer({ src: activeBrand.heroDetailImage, alt: `${activeBrand.label} detail`, title: `${activeBrand.label} detail`, meta: 'Click outside or press Escape to close.' })"
                @keydown.enter.prevent="openImageViewer({ src: activeBrand.heroDetailImage, alt: `${activeBrand.label} detail`, title: `${activeBrand.label} detail`, meta: 'Click outside or press Escape to close.' })"
                @keydown.space.prevent="openImageViewer({ src: activeBrand.heroDetailImage, alt: `${activeBrand.label} detail`, title: `${activeBrand.label} detail`, meta: 'Click outside or press Escape to close.' })"
              />
              <img
                class="hero-car interactive-media"
                :class="{ 'vertical-orient': activeBrand.isVertical }"
                :src="activeBrand.heroImage"
                :alt="activeBrand.imageAlt"
                loading="eager"
                decoding="async"
                fetchpriority="high"
                role="button"
                tabindex="0"
                :aria-label="`View ${activeBrand.imageAlt}`"
                @click="openImageViewer({ src: activeBrand.heroImage, alt: activeBrand.imageAlt, title: activeBrand.imageAlt, meta: 'Click outside or press Escape to close.' })"
                @keydown.enter.prevent="openImageViewer({ src: activeBrand.heroImage, alt: activeBrand.imageAlt, title: activeBrand.imageAlt, meta: 'Click outside or press Escape to close.' })"
                @keydown.space.prevent="openImageViewer({ src: activeBrand.heroImage, alt: activeBrand.imageAlt, title: activeBrand.imageAlt, meta: 'Click outside or press Escape to close.' })"
              />
              <div class="hero-stat hero-stat-left">
                <strong>{{ activeBrand.statTitle }}</strong>
                <span>{{ activeBrand.statText }}</span>
              </div>
              <div class="hero-stat hero-stat-right">
                <strong>{{ activeBrand.sideStatTitle }}</strong>
                <span>{{ activeBrand.sideStatText }}</span>
              </div>
            </div>
          </Transition>
        </div>

        <div class="brand-strip" role="tablist" aria-label="Brand gallery">
          <button
            v-for="logo in brandLogos"
            :key="logo.id"
            class="logo-pill"
            :class="{ active: logo.id === activeBrandId }"
            type="button"
            :aria-pressed="logo.id === activeBrandId"
            :style="{ '--brand-color': logo.accent }"
            @click="selectBrand(logo.id)"
          >
            <img
              :src="logo.src"
              :alt="logo.alt"
              :loading="logo.id === activeBrandId ? 'eager' : 'lazy'"
              decoding="async"
              :fetchpriority="logo.id === activeBrandId ? 'high' : 'low'"
            />
          </button>
        </div>
      </section>

      <section id="stories-intro" class="intro-band">
        <p class="eyebrow">The Digital Exhibition</p>
        <div class="intro-layout">
          <h2>A journey through masterpieces, where technology meets the dramatic art of mechanical engineering.</h2>
          <p>
            Mỗi chương là một trải nghiệm thị giác khác biệt. Chúng tôi sử dụng bố cục magazine cao cấp kết hợp với nhịp chuyển động mượt mà để kể về linh hồn của những cỗ máy tốc độ, thay vì chỉ là những thông số khô khan trên trang giấy.
          </p>
        </div>
      </section>

      <section
        v-for="(story, index) in stories"
        :id="story.id"
        :key="story.id"
        :ref="(el) => registerStorySection(el, index)"
        class="story-section"
        :class="[`align-${story.align}`, mobileStoryPatternClass(index)]"
        :style="storyTheme(story)"
      >
        <div class="story-grid">
          <article class="story-copy">
            <p class="section-badge">{{ story.chapter }}</p>
            <p class="story-brand">{{ story.brand }}</p>
            <h2 class="story-title">{{ story.model }}</h2>
            <p class="story-quote">{{ story.headline }}</p>
            <p class="story-description">{{ story.story }}</p>
            <p class="story-note">{{ story.note }}</p>
            <div class="spec-row">
              <span v-for="spec in story.specs" :key="spec" class="spec-chip">{{ spec }}</span>
            </div>
            <button class="text-btn" type="button" @click="scrollToSection(nextSection(index))">
              {{ index === stories.length - 1 ? 'Go to contact' : 'Next chapter' }}
            </button>
          </article>

          <div class="story-media-wrap">
            <div class="story-media-shell tilt-surface" @mousemove="onTiltMove" @mouseleave="onTiltLeave">
              <div class="story-light"></div>
              <div class="story-number">{{ story.serial }}</div>
              <div
                class="story-frame secondary interactive-frame"
                role="button"
                tabindex="0"
                :aria-label="`View ${story.brand} detail image`"
                @click="openImageViewer({ src: story.secondaryImage, alt: `${story.brand} detail`, title: `${story.brand} detail`, meta: story.mood })"
                @keydown.enter.prevent="openImageViewer({ src: story.secondaryImage, alt: `${story.brand} detail`, title: `${story.brand} detail`, meta: story.mood })"
                @keydown.space.prevent="openImageViewer({ src: story.secondaryImage, alt: `${story.brand} detail`, title: `${story.brand} detail`, meta: story.mood })"
              >
                <img
                  v-if="isStoryMediaReady(index)"
                  :class="{ 'vertical-orient-story': story.isVertical }"
                  :src="story.secondaryImage"
                  :alt="`${story.brand} detail`"
                  loading="lazy"
                  decoding="async"
                  fetchpriority="low"
                />
                <div v-else class="story-image-placeholder" aria-hidden="true"></div>
              </div>
              <div
                class="story-frame primary interactive-frame"
                role="button"
                tabindex="0"
                :aria-label="`View ${story.brand} ${story.model}`"
                @click="openImageViewer({ src: story.image, alt: `${story.brand} ${story.model}`, title: `${story.brand} ${story.model}`, meta: story.caption })"
                @keydown.enter.prevent="openImageViewer({ src: story.image, alt: `${story.brand} ${story.model}`, title: `${story.brand} ${story.model}`, meta: story.caption })"
                @keydown.space.prevent="openImageViewer({ src: story.image, alt: `${story.brand} ${story.model}`, title: `${story.brand} ${story.model}`, meta: story.caption })"
              >
                <img
                  v-if="isStoryMediaReady(index)"
                  :class="{ 'vertical-orient-story': story.isVertical }"
                  :src="story.image"
                  :alt="`${story.brand} ${story.model}`"
                  loading="lazy"
                  decoding="async"
                  fetchpriority="low"
                />
                <div v-else class="story-image-placeholder" aria-hidden="true"></div>
              </div>
              <div class="story-caption">
                <span>{{ story.caption }}</span>
                <small>{{ story.mood }}</small>
              </div>
            </div>
          </div>
        </div>
      </section>

      <section id="reserve" class="closing-panel tilt-surface" @mousemove="onTiltMove" @mouseleave="onTiltLeave">
        <div class="closing-copy">
          <p class="eyebrow">Contact</p>
          <p class="contact-kicker">Unique, creative UI design and front-end development, shaped around real brand goals.</p>
          <h2 class="contact-display">Open to projects involving landing pages, websites, portfolios, and product-focused marketing sites.</h2>
          <p class="contact-lede">
            If you need a website that feels polished, clear, and aligned with your brand, I can take it from concept
            to final build.
          </p>
          <div class="contact-tags" aria-label="Service details">
            <span>Freelance projects</span>
            <span>Remote worldwide</span>
            <span>Fast revision flow</span>
          </div>
        </div>
        <div class="contact-layout">
          <a class="contact-spotlight" href="tel:0981003682">
            <span class="contact-spotlight-label">Direct line</span>
            <strong class="contact-spotlight-value">0981003682</strong>
            <p class="contact-spotlight-copy">
              Quick calls for redesigns, launch polish, and fast fixes.
            </p>
            <span class="contact-spotlight-cta">Call to start a project</span>
          </a>
          <div class="contact-connect">
            <p class="contact-connect-label">Connect with me</p>
            <div class="contact-connect-buttons">
              <a class="contact-connect-btn" href="mailto:ndtuanawh@gmail.com">
                <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                  <rect x="2" y="4" width="20" height="16" rx="2"/>
                  <path d="m2 7 10 7 10-7"/>
                </svg>
                Email
              </a>
              <a class="contact-connect-btn" href="https://github.com/Awh-216" target="_blank">
                <svg width="20" height="20" viewBox="0 0 24 24" fill="currentColor">
                  <path d="M12 2C6.477 2 2 6.477 2 12c0 4.42 2.865 8.17 6.839 9.49.5.092.682-.217.682-.482 0-.237-.008-.866-.013-1.7-2.782.603-3.369-1.34-3.369-1.34-.454-1.156-1.11-1.463-1.11-1.463-.908-.62.069-.608.069-.608 1.003.07 1.531 1.03 1.531 1.03.892 1.529 2.341 1.087 2.91.831.092-.646.35-1.086.636-1.336-2.22-.253-4.555-1.11-4.555-4.943 0-1.091.39-1.984 1.029-2.683-.103-.253-.446-1.27.098-2.647 0 0 .84-.269 2.75 1.025A9.578 9.578 0 0112 6.836c.85.004 1.705.114 2.504.336 1.909-1.294 2.747-1.025 2.747-1.025.546 1.377.203 2.394.1 2.647.64.699 1.028 1.592 1.028 2.683 0 3.842-2.339 4.687-4.566 4.935.359.309.678.919.678 1.852 0 1.336-.012 2.415-.012 2.743 0 .267.18.578.688.48C19.138 20.167 22 16.418 22 12c0-5.523-4.477-10-10-10z"/>
                </svg>
                GitHub
              </a>
              <a class="contact-connect-btn" href="https://t.me/awh_216" target="_blank">
                <svg width="20" height="20" viewBox="0 0 24 24" fill="currentColor">
                  <path d="M12 2C6.48 2 2 6.48 2 12s4.48 10 10 10 10-4.48 10-10S17.52 2 12 2zm4.64 6.8c-.15 1.58-.8 5.42-1.13 7.19-.14.75-.42 1-.68 1.03-.58.05-1.02-.38-1.58-.75-.88-.58-1.38-.94-2.23-1.5-.99-.65-.35-1.01.22-1.59.15-.15 2.71-2.48 2.76-2.69a.2.2 0 00-.05-.18c-.06-.05-.14-.03-.21-.02-.09.02-1.49.95-4.22 2.79-.4.27-.76.41-1.08.4-.36-.01-1.04-.2-1.55-.37-.63-.2-1.12-.31-1.08-.66.02-.18.27-.36.74-.55 2.92-1.27 4.86-2.1１ 5.83-2.5１ ２.７８-１.１６ ３．３５-１．３６ ３．７３-１．３６．０８ ０．２７．０２．３９．１２．１．０８．１３．１９．１４．２７－．０１．０６．０１．２４ ０．３８z"/>
                </svg>
                Telegram
              </a>
            </div>
          </div>
          <div class="contact-bento">
            <div class="contact-detail">
              <span class="contact-label">Project Fit</span>
              <strong class="contact-value">Landing pages and brand sites</strong>
              <span class="contact-meta">Concept to responsive build.</span>
            </div>
            <div class="contact-detail">
              <span class="contact-label">Work Style</span>
              <strong class="contact-value">Clean visuals and quick iteration</strong>
              <span class="contact-meta">Clear process, practical execution.</span>
            </div>
            <div class="contact-detail">
              <span class="contact-label">Availability</span>
              <strong class="contact-value">Open for selected freelance work</strong>
              <span class="contact-meta">Short sprints or full redesigns.</span>
            </div>
            <div class="contact-detail">
              <span class="contact-label">Timezone</span>
              <strong class="contact-value">GMT+7, remote-friendly</strong>
              <span class="contact-meta">Easy with local and global clients.</span>
            </div>
          </div>
        </div>
      </section>
    </div>

    <Transition name="lightbox-fade">
      <div v-if="selectedImage" class="image-lightbox" role="dialog" aria-modal="true" :aria-label="selectedImage.title">
        <button class="image-lightbox-backdrop" type="button" aria-label="Close image viewer" @click="closeImageViewer"></button>
        <div class="image-lightbox-panel">
          <button class="image-lightbox-close" type="button" @click="closeImageViewer">Close</button>
          
          <!-- Navigation buttons - chỉ hiện khi có gallery context -->
          <button 
            v-if="currentGalleryImages.length > 1 && currentImageIndex > 0 && currentGalleryBrand"
            class="image-nav-btn image-nav-prev" 
            type="button" 
            aria-label="Previous image"
            @click="navigateImage(-1)"
          >
            <svg width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
              <path d="M15 18l-6-6 6-6"/>
            </svg>
          </button>
          
          <button 
            v-if="currentGalleryImages.length > 1 && currentImageIndex < currentGalleryImages.length - 1 && currentGalleryBrand"
            class="image-nav-btn image-nav-next" 
            type="button" 
            aria-label="Next image"
            @click="navigateImage(1)"
          >
            <svg width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
              <path d="M9 18l6-6-6-6"/>
            </svg>
          </button>
          
          <figure class="image-lightbox-figure">
            <img class="image-lightbox-image" :src="selectedImage.src" :alt="selectedImage.alt" />
            <figcaption class="image-lightbox-caption">
              <strong>{{ selectedImage.title }}</strong>
              <span>{{ selectedImage.meta }}</span>
            </figcaption>
          </figure>
        </div>
      </div>
    </Transition>

    <Transition name="lightbox-fade">
      <div v-if="galleryOpen" class="gallery-lightbox" role="dialog" aria-modal="true" :aria-label="`${currentGalleryBrand} Gallery`" @click.self="closeGallery">
        <button class="image-lightbox-backdrop" type="button" aria-label="Close gallery" @click="closeGallery"></button>
        <div class="gallery-panel" @click.stop>
          <div class="gallery-header">
            <h2>{{ currentGalleryBrand }} Gallery</h2>
            <button class="image-lightbox-close" type="button" @click="closeGallery">Close</button>
          </div>
          <div class="gallery-grid">
            <div
              v-for="(img, index) in currentGalleryImages"
              :key="index"
              class="gallery-item"
              role="button"
              tabindex="0"
              :aria-label="`View ${currentGalleryBrand} image ${index + 1}`"
              @click="openImageFromGallery(img, index)"
              @keydown.enter.prevent="openImageFromGallery(img, index)"
              @keydown.space.prevent="openImageFromGallery(img, index)"
            >
              <img :src="img" :alt="`${currentGalleryBrand} ${index + 1}`" loading="lazy" />
            </div>
          </div>
        </div>
      </div>
    </Transition>
  </main>
</template>

<script setup>
import { computed, nextTick, onMounted, onUnmounted, ref, watch } from 'vue'
import { gsap } from 'gsap'
import { ScrollTrigger } from 'gsap/ScrollTrigger'
import Lenis from 'lenis'

gsap.registerPlugin(ScrollTrigger)

const main = ref(null)
const siteNav = ref(null)
const heroCard = ref(null)
const storySections = ref([])
const isMobileMenuOpen = ref(false)
const selectedImage = ref(null)
const galleryOpen = ref(false)
const currentGalleryBrand = ref('')
const currentGalleryImages = ref([])
const currentImageIndex = ref(0)
const isDarkMode = ref(false)
const readyStoryMedia = ref({ 0: true })

const THEME_STORAGE_KEY = 'atelier-drive-theme'

const asset = (path) => `${import.meta.env.BASE_URL}${path}`

const brandGalleries = {
  bmw: [
    asset('assets/images/BMW/1.png'),
    asset('assets/images/BMW/2.png'),
    asset('assets/images/BMW/3.png'),
  ],
  lamborghini: [
    asset('assets/images/lamborghini/4.png'),
    asset('assets/images/lamborghini/5.png'),
    asset('assets/images/lamborghini/6.png'),
  ],
  audi: [
    asset('assets/images/audi/7.png'),
    asset('assets/images/audi/8.png'),
    asset('assets/images/audi/9.png'),
  ],
  mercedes: [
    asset('assets/images/mercedes/10.png'),
    asset('assets/images/mercedes/11.png'),
    asset('assets/images/mercedes/12.png'),
  ],
  porsche: [
    asset('assets/images/porscher/13.png'),
    asset('assets/images/porscher/14.png'),
    asset('assets/images/porscher/15.png'),
  ],
}

const brandLogos = [
  {
    id: 'bmw',
    label: 'BMW',
    src: asset('assets/images/logo/bmw.png'),
    alt: 'BMW logo',
    eyebrow: 'Precision Engineering',
    heroTitle: 'BMW: The pure thrill of mechanical soul.',
    heroDescription: 'Kỹ nghệ chính xác gặp gỡ bản năng tốc độ. Một kết cấu tập trung hoàn toàn vào người lái, nơi mọi chi tiết đều được siết chặt để tạo ra cảm giác kiểm soát tuyệt đối.',
    heroImage: asset('assets/images/BMW/3.png'),
    heroDetailImage: asset('assets/images/BMW/1.png'),
    imageAlt: 'BMW M4 CSL',
    topTag: 'Engineering Masterclass',
    bottomTag: 'Carbon-fiber obsession for the track',
    statTitle: 'M-Performance',
    statText: 'Sự cân bằng hoàn hảo giữa sức mạnh thô và độ chính xác kỹ thuật.',
    sideStatTitle: 'Driver First',
    sideStatText: 'Khoang lái được thiết kế để trở thành phần mở rộng của cơ thể.',
    storyId: 'bmw-story',
    facts: [
      { value: '543 HP', label: 'In-line six punch' },
      { value: '3.7s', label: '0-100 km/h' },
      { value: 'CSL', label: 'Track soul' },
    ],
    accent: '#6a93ff',
    isVertical: true,
  },
  {
    id: 'lamborghini',
    label: 'Lamborghini',
    src: asset('assets/images/logo/lamborghini.png'),
    alt: 'Lamborghini logo',
    eyebrow: 'Italian Adrenaline',
    heroTitle: 'Lamborghini: A storm of angular geometry.',
    heroDescription: 'Không chỉ là một cỗ máy, đó là một tuyên ngôn. Thân xe là những đường cắt kịch tính, phá vỡ mọi quy tắc khí động học để đạt tới sự bùng nổ của thị giác.',
    heroImage: asset('assets/images/lamborghini/6.png'),
    heroDetailImage: asset('assets/images/lamborghini/4.png'),
    imageAlt: 'Lamborghini Revuelto',
    topTag: 'Cinematic Presence',
    bottomTag: 'Aggressive lines, unmistakable DNA',
    statTitle: 'V12 Energy',
    statText: 'Tiếng gầm của động cơ V12 là bản thánh ca của sức mạnh vô tận.',
    sideStatTitle: 'Hypershape',
    sideStatText: 'Mỗi góc cạnh đều được tính toán để tạo ra áp lực thị giác mạnh mẽ.',
    storyId: 'lamborghini-story',
    facts: [
      { value: '1015 CV', label: 'Hybrid V12' },
      { value: '2.5s', label: '0-100 km/h' },
      { value: 'Italy', label: 'Born in Sant’Agata' },
    ],
    accent: '#ff6b2e',
  },
  {
    id: 'audi',
    label: 'Audi',
    src: asset('assets/images/logo/audi.png'),
    alt: 'Audi logo',
    eyebrow: 'Vorsprung durch Technik',
    heroTitle: 'Audi: The silent architecture of progress.',
    heroDescription: 'Định nghĩa lại tương lai bằng ngôn ngữ tối giản. Audi e-tron là sự giao thoa giữa công nghệ thuần điện và sự sang trọng kín tiếng, nơi im lặng là quyền lực.',
    heroImage: asset('assets/images/audi/9.png'),
    heroDetailImage: asset('assets/images/audi/7.png'),
    imageAlt: 'Audi RS e-tron GT',
    topTag: 'Electric Grand Tourer',
    bottomTag: 'Minimalist form, maximum momentum',
    statTitle: 'Quattro DNA',
    statText: 'Hệ dẫn động huyền thoại giờ đây được tái sinh bằng năng lượng điện.',
    sideStatTitle: 'Future Poise',
    sideStatText: 'Sự tĩnh lặng sang trọng bao bọc lấy một trái tim tốc độ.',
    storyId: 'audi-story',
    facts: [
      { value: 'RS', label: 'Electric sport' },
      { value: '800V', label: 'Speed charging' },
      { value: 'Design', label: 'Liquid metal' },
    ],
    accent: '#8c7bff',
  },
  {
    id: 'mercedes',
    label: 'Mercedes',
    src: asset('assets/images/logo/mercedes.png'),
    alt: 'Mercedes logo',
    eyebrow: 'Modern Prestige',
    heroTitle: 'Mercedes: The pinnacle of quiet authority.',
    heroDescription: 'Nơi sự êm ái trở thành một nghi thức. Mỗi chuyến đi là một trải nghiệm trong không gian Lounge thượng hạng, tách biệt hoàn toàn với thế giới ồn ào bên ngoài.',
    heroImage: asset('assets/images/mercedes/12.png'),
    heroDetailImage: asset('assets/images/mercedes/10.png'),
    imageAlt: 'Mercedes-Maybach EQS SUV',
    topTag: 'Ultimate Comfort',
    bottomTag: 'A sanctuary of high-tech luxury',
    statTitle: 'Maybach Mood',
    statText: 'Sự hoàn thiện đạt tới mức độ thủ công tinh xảo nhất của nhân loại.',
    sideStatTitle: 'Silent Luxury',
    sideStatText: 'Sức mạnh lớn lao được vận hành một cách khẽ khàng nhất.',
    storyId: 'mercedes-story',
    facts: [
      { value: 'Lounge', label: 'Interior mood' },
      { value: 'EQS', label: 'Electric flagship' },
      { value: 'Crafted', label: 'Detail focus' },
    ],
    accent: '#3db8b2',
  },
  {
    id: 'porsche',
    label: 'Porsche',
    src: asset('assets/images/logo/porscher.png'),
    alt: 'Porsche logo',
    eyebrow: 'The 911 Heritage',
    heroTitle: 'Porsche: A timeless line, an eternal icon.',
    heroDescription: 'Đường cong không bao giờ lỗi thời. 911 là biểu tượng của sự tự do, một sự kết hợp không thể thay thế giữa di sản thể thao và sự thực dụng mỗi ngày.',
    heroImage: asset('assets/images/porscher/15.png'),
    heroDetailImage: asset('assets/images/porscher/14.png'),
    imageAlt: 'Porsche 911 Cabriolet',
    topTag: 'Iconic Silhouette',
    bottomTag: 'Open-top freedom in every curve',
    statTitle: '911 Soul',
    statText: 'Cảm giác lái là tôn chỉ duy nhất tồn tại xuyên suốt nhiều thập kỷ.',
    sideStatTitle: 'Pure Porsche',
    sideStatText: 'Ranh giới giữa xe đua và xe đường phố bị xóa nhòa.',
    storyId: 'porsche-story',
    facts: [
      { value: '911', label: 'Pure legacy' },
      { value: 'Flat-6', label: 'Engine pulse' },
      { value: 'Open Air', label: 'Convertible flow' },
    ],
    accent: '#ef867f',
  },
]

const activeBrandId = ref('bmw')
const activeBrand = computed(
  () => brandLogos.find((brand) => brand.id === activeBrandId.value) ?? brandLogos[0],
)
const themeButtonLabel = computed(() => (isDarkMode.value ? 'Switch to light mode' : 'Switch to dark mode'))

const stories = [
  {
    id: 'bmw-story',
    serial: '01',
    chapter: 'Chapter 01',
    brand: 'The M-Genetics',
    model: 'BMW M4 CSL',
    headline: 'Một đường cắt lạnh lùng và tỉnh táo giữa bản hòa tấu đô thị.',
    story: 'M4 CSL không chỉ là một chiếc xe, nó là một thí nghiệm về sự chuẩn xác. Khi nắp capo carbon phản chiếu ánh nắng bình minh, mọi thứ dường như ngưng đọng lại để nhường chỗ cho khối động cơ Twin-Turbo đang trực chờ bùng nổ. Đây là nơi kỹ thuật cơ khí trở thành một lời khẳng định về bản ngã.',
    note: 'Siết chặt dây an toàn, và để cảm giác lái làm chủ cuộc chơi.',
    specs: ['543 HP', '3.7s 0-100', 'M-Performance'],
    caption: 'Cold Precision / M-Blood',
    mood: 'The sharpest blade in a tailored suit',
    image: asset('assets/images/BMW/3.png'),
    secondaryImage: asset('assets/images/BMW/1.png'),
    isVertical: true,
    accent: '#6a93ff',
    accentSoft: 'rgba(106, 147, 255, 0.18)',
    align: 'right',
  },
  {
    id: 'lamborghini-story',
    serial: '02',
    chapter: 'Chapter 02',
    brand: 'The Predator DNA',
    model: 'Revuelto V12',
    headline: 'Khi hình khối không chỉ để nhìn, mà để xuyên thủng không gian.',
    story: 'Revuelto là một cơn bão hình học. Thân xe căng tràn sự đe dọa, góc cạnh như một tác phẩm điêu khắc mang năng lượng từ tương lai. Với động cơ Hybrid V12, chiếc xe này biến mỗi lần nhấn ga thành một cú hích đưa bạn rời khỏi thực tại để bước vào kỷ nguyên của tốc độ thuần khiết.',
    note: 'Dành cho những người muốn trở thành tâm điểm của mọi ánh nhìn.',
    specs: ['1015 CV', 'V12 Hybrid', 'Absolute Adrenaline'],
    caption: 'Geometry of Violence',
    mood: 'Aggressive, loud, and cinematic',
    image: asset('assets/images/lamborghini/6.png'),
    secondaryImage: asset('assets/images/lamborghini/4.png'),
    accent: '#ff6b2e',
    accentSoft: 'rgba(255, 107, 46, 0.18)',
    align: 'left',
  },
  {
    id: 'audi-story',
    serial: '03',
    chapter: 'Chapter 03',
    brand: 'The Future Poise',
    model: 'RS e-tron GT',
    headline: 'Sức mạnh lớn lao giờ đây được vận hành trong im lặng tuyệt đối.',
    story: 'Audi định nghĩa lại sự sang trọng bằng sự tĩnh lặng. Chiếc RS e-tron GT lướt đi như một dòng chảy kim loại lỏng, mang trong mình công nghệ Quattro huyền thoại nhưng được tinh chỉnh bởi năng lượng điện 800V. Khi sự tối giản đạt tới đỉnh cao, nó trở thành một loại quyền lực kín tiếng.',
    note: 'Tương lai không ồn ào, tương lai là sự tinh tế và mượt mà.',
    specs: ['934 Nm', 'Electric Poise', '800V Architecture'],
    caption: 'Liquid Metal / Silent Force',
    mood: 'The art of quiet momentum',
    image: asset('assets/images/audi/9.png'),
    secondaryImage: asset('assets/images/audi/7.png'),
    accent: '#8c7bff',
    accentSoft: 'rgba(140, 123, 255, 0.18)',
    align: 'right',
  },
  {
    id: 'mercedes-story',
    serial: '04',
    chapter: 'Chapter 04',
    brand: 'The Maybach Sanctuary',
    model: 'EQS SUV Maybach',
    headline: 'Nơi sự xa hoa không còn giới hạn, và thời gian dường như dừng lại.',
    story: 'Khoang xe Maybach là một thánh đường của sự tĩnh lặng. Tại đây, mọi bề mặt đều được chạm khắc thủ công, kết nối công nghệ Hyperscreen tiên phong với truyền thống chế tác hàng đầu. Đây không đơn thuần là một phương tiện, đó là một không gian nghỉ ngơi di động thượng hạng nhất.',
    note: 'Chương dành cho những người tìm kiếm sự hoàn mỹ và êm ái tuyệt đối.',
    specs: ['Executive Lounge', 'Hyperscreen', 'AirMatic'],
    caption: 'Architect of Comfort',
    mood: 'Soft power / Sovereign grace',
    image: asset('assets/images/mercedes/12.png'),
    secondaryImage: asset('assets/images/mercedes/10.png'),
    accent: '#3db8b2',
    accentSoft: 'rgba(61, 184, 178, 0.18)',
    align: 'left',
  },
  {
    id: 'porsche-story',
    serial: '05',
    chapter: 'Chapter 05',
    brand: 'The 911 Eternal',
    model: '911 Cabriolet',
    headline: 'Một đường cong vĩnh cửu, nối liền quá khứ và tương lai.',
    story: 'Porsche 911 không cố gắng thay đổi thế giới, nó tạo ra thế giới của riêng mình. Với mui trần mở ra bầu trời tự do, âm thanh của động cơ Flat-6 đặt sau là nhịp đập của sự tự do thực sự. Một sự phối hợp kinh điển khiến mọi chuyến đi trở thành một kỷ niệm khó quên.',
    note: 'Kết thúc hành trình bằng một biểu tượng không bao giờ phai mờ.',
    specs: ['Timeless Design', 'Flat-6 Soul', 'Pure Freedom'],
    caption: 'Iconic Flow / Open Heart',
    mood: 'Classic restraint with silver light',
    image: asset('assets/images/porscher/15.png'),
    secondaryImage: asset('assets/images/porscher/13.png'),
    accent: '#ef867f',
    accentSoft: 'rgba(239, 134, 127, 0.18)',
    align: 'right',
  },
]

let lenis
let lenisScrollHandler
let gsapTicker
let pointerHandler
let resizeHandler
let keydownHandler
let storyMediaObserver
let idlePreloadHandle
let idlePreloadUsesIdleCallback = false

const getNavOffset = () => (siteNav.value?.offsetHeight ?? 82) + 62

const registerStorySection = (el, index) => {
  if (el) storySections.value[index] = el
}

const storyTheme = (story) => ({
  '--accent': story.accent,
  '--accent-soft': story.accentSoft,
})

const isStoryMediaReady = (index) => Boolean(readyStoryMedia.value[index])

const nextSection = (index) => (index === stories.length - 1 ? 'reserve' : stories[index + 1].id)

const mobileStoryPatternClass = (index) => (Math.floor(index / 2) % 2 === 0 ? 'mobile-pair-right' : 'mobile-pair-left')

const isMobileViewport = () => window.innerWidth <= 820

const isViewSourceShortcut = (event) => (event.ctrlKey || event.metaKey) && event.key.toLowerCase() === 'u'

const closeMobileMenu = () => {
  isMobileMenuOpen.value = false
}

const toggleMobileMenu = () => {
  if (!isMobileViewport()) return
  isMobileMenuOpen.value = !isMobileMenuOpen.value
}

const selectBrand = (brandId) => {
  activeBrandId.value = brandId
}

const preloadImage = (src, fetchPriority = 'auto') => {
  if (!src || document.head.querySelector(`link[rel="preload"][href="${src}"]`)) return
  const link = document.createElement('link')
  link.rel = 'preload'
  link.as = 'image'
  link.href = src
  if (fetchPriority !== 'auto') link.setAttribute('fetchpriority', fetchPriority)
  document.head.appendChild(link)
}

const preloadBrandAssets = (brand, fetchPriority = 'low') => {
  if (!brand) return
  preloadImage(brand.heroImage, fetchPriority)
  preloadImage(brand.heroDetailImage, 'low')
}

const warmNearbyBrands = (brandId) => {
  const currentIndex = brandLogos.findIndex((brand) => brand.id === brandId)
  if (currentIndex === -1) return
  ;[brandLogos[currentIndex - 1], brandLogos[currentIndex + 1]].forEach((brand) => preloadBrandAssets(brand, 'low'))
}

const clearBrandWarmup = () => {
  if (typeof window === 'undefined' || idlePreloadHandle == null) return
  if (idlePreloadUsesIdleCallback && typeof window.cancelIdleCallback === 'function') {
    window.cancelIdleCallback(idlePreloadHandle)
  } else {
    window.clearTimeout(idlePreloadHandle)
  }
  idlePreloadHandle = null
  idlePreloadUsesIdleCallback = false
}

const scheduleBrandWarmup = (brandId) => {
  if (typeof window === 'undefined') return
  clearBrandWarmup()

  const runWarmup = () => warmNearbyBrands(brandId)

  if (typeof window.requestIdleCallback === 'function') {
    idlePreloadUsesIdleCallback = true
    idlePreloadHandle = window.requestIdleCallback(runWarmup, { timeout: 1200 })
    return
  }

  idlePreloadUsesIdleCallback = false
  idlePreloadHandle = window.setTimeout(runWarmup, 350)
}

const markStoryMediaReady = (index) => {
  if (index < 0 || index >= stories.length || readyStoryMedia.value[index]) return
  readyStoryMedia.value = {
    ...readyStoryMedia.value,
    [index]: true,
  }
}

const markStoryRangeReady = (index) => {
  markStoryMediaReady(index - 1)
  markStoryMediaReady(index)
  markStoryMediaReady(index + 1)
}

const initStoryMediaObserver = () => {
  markStoryRangeReady(0)

  if (typeof window === 'undefined' || !('IntersectionObserver' in window)) {
    stories.forEach((_, index) => markStoryMediaReady(index))
    return
  }

  storyMediaObserver = new IntersectionObserver(
    (entries) => {
      entries.forEach((entry) => {
        if (!entry.isIntersecting) return
        const index = Number(entry.target.getAttribute('data-story-index'))
        markStoryRangeReady(index)
        storyMediaObserver?.unobserve(entry.target)
      })
    },
    {
      rootMargin: '320px 0px',
    },
  )

  storySections.value.forEach((section, index) => {
    if (!section) return
    section.setAttribute('data-story-index', `${index}`)
    storyMediaObserver.observe(section)
  })
}

const applyTheme = () => {
  document.body.dataset.theme = isDarkMode.value ? 'dark' : 'light'
}

const toggleTheme = () => {
  isDarkMode.value = !isDarkMode.value
}

const openImageViewer = (image) => {
  selectedImage.value = image
  document.body.style.overflow = 'hidden'
}

const closeImageViewer = () => {
  selectedImage.value = null
  document.body.style.overflow = ''
  // Reset gallery context khi đóng lightbox nếu không đang trong gallery
  if (!galleryOpen.value) {
    currentGalleryImages.value = []
    currentImageIndex.value = 0
    currentGalleryBrand.value = ''
  }
}

const openBrandGallery = (brandId) => {
  currentGalleryBrand.value = brandLogos.find(b => b.id === brandId)?.label || brandId.toUpperCase()
  currentGalleryImages.value = brandGalleries[brandId] || []
  galleryOpen.value = true
  document.body.style.overflow = 'hidden'
}

const closeGallery = () => {
  galleryOpen.value = false
  document.body.style.overflow = ''
}

const openImageFromGallery = (img, index) => {
  // Lưu thông tin ảnh và index
  currentImageIndex.value = index
  const imageData = {
    src: img,
    alt: `${currentGalleryBrand.value} ${index + 1}`,
    title: currentGalleryBrand.value,
    meta: `Image ${index + 1} of ${currentGalleryImages.value.length}`
  }
  
  // Đóng gallery ngay lập tức
  galleryOpen.value = false
  
  // Đợi animation đóng gallery hoàn tất
  setTimeout(() => {
    document.body.style.overflow = 'hidden'
    openImageViewer(imageData)
  }, 350)
}

const navigateImage = (direction) => {
  const newIndex = currentImageIndex.value + direction
  if (newIndex >= 0 && newIndex < currentGalleryImages.value.length) {
    currentImageIndex.value = newIndex
    const img = currentGalleryImages.value[newIndex]
    selectedImage.value = {
      src: img,
      alt: `${currentGalleryBrand.value} ${newIndex + 1}`,
      title: currentGalleryBrand.value,
      meta: `Image ${newIndex + 1} of ${currentGalleryImages.value.length}`
    }
  }
}

const navigateToSection = (id) => {
  if (lenis) lenis.start()
  closeMobileMenu()
  requestAnimationFrame(() => scrollToSection(id))
}

const scrollToSection = (id) => {
  const target = document.getElementById(id)
  if (!target) return
  const navOffset = getNavOffset()
  if (lenis) {
    lenis.scrollTo(target, { duration: 1.2, offset: -navOffset })
    return
  }
  const top = target.getBoundingClientRect().top + window.scrollY - navOffset
  window.scrollTo({ top, behavior: 'smooth' })
}

const onTiltMove = (event) => {
  if (window.matchMedia('(hover: none)').matches) return
  const element = event.currentTarget
  const rect = element.getBoundingClientRect()
  const relativeX = event.clientX - rect.left
  const relativeY = event.clientY - rect.top
  const rotateY = gsap.utils.mapRange(0, rect.width, -10, 10, relativeX)
  const rotateX = gsap.utils.mapRange(0, rect.height, 9, -9, relativeY)
  const shiftX = gsap.utils.mapRange(0, rect.width, -4, 4, relativeX)
  const shiftY = gsap.utils.mapRange(0, rect.height, -6, 6, relativeY)

  gsap.to(element, {
    '--rotate-x': `${rotateX}deg`,
    '--rotate-y': `${rotateY}deg`,
    '--shift-x': `${shiftX}px`,
    '--shift-y': `${shiftY}px`,
    duration: 0.25,
    ease: 'power2.out',
    overwrite: true,
  })
}

const onTiltLeave = (event) => {
  gsap.to(event.currentTarget, {
    '--rotate-x': '0deg',
    '--rotate-y': '0deg',
    '--shift-x': '0px',
    '--shift-y': '0px',
    duration: 0.45,
    ease: 'power3.out',
  })
}

const initLenis = () => {
  lenis = new Lenis({
    duration: 1.15,
    smoothWheel: true,
    gestureOrientation: 'vertical',
  })

  lenisScrollHandler = () => ScrollTrigger.update()
  lenis.on('scroll', lenisScrollHandler)

  gsapTicker = (time) => {
    lenis.raf(time * 1000)
  }

  gsap.ticker.add(gsapTicker)
  gsap.ticker.lagSmoothing(0)
}

const initAmbientPointer = () => {
  pointerHandler = (event) => {
    if (!main.value) return
    const rect = main.value.getBoundingClientRect()
    main.value.style.setProperty('--pointer-x', `${event.clientX - rect.left}px`)
    main.value.style.setProperty('--pointer-y', `${event.clientY - rect.top}px`)
  }

  main.value?.addEventListener('pointermove', pointerHandler)
}

const initAnimations = () => {
  gsap.from('.site-nav', { y: -24, opacity: 0, duration: 0.8, ease: 'power3.out' })
  gsap.from('.hero-copy > *', { y: 42, opacity: 0, duration: 0.95, stagger: 0.1, ease: 'power3.out', delay: 0.1 })
  gsap.from('.hero-visual > *', { y: 32, opacity: 0, duration: 1, stagger: 0.08, ease: 'power3.out', delay: 0.2 })
  // Hiệu ứng mở đầu tối giản, bỏ tất cả những gì gây lỗi cho logo

  gsap.from('.intro-band .eyebrow, .intro-band h2, .intro-band p:last-child', {
    y: 40,
    opacity: 0,
    duration: 0.9,
    stagger: 0.12,
    ease: 'power3.out',
    scrollTrigger: { trigger: '.intro-band', start: 'top 78%' },
  })

  storySections.value.forEach((section) => {
    if (!section) return
    const copy = section.querySelector('.story-copy')
    const mediaWrap = section.querySelector('.story-media-wrap')
    const secondary = section.querySelector('.story-frame.secondary')
    const chips = section.querySelectorAll('.spec-chip')

    const isAlignRight = section.classList.contains('align-right')

    gsap.from(copy, {
      x: isAlignRight ? 120 : -120,
      opacity: 0,
      duration: 1.2,
      ease: 'power4.out',
      scrollTrigger: { trigger: section, start: 'top 82%' },
    })

    gsap.from(mediaWrap, {
      y: -140,
      opacity: 0,
      duration: 1.4,
      ease: 'power4.out',
      scrollTrigger: { trigger: section, start: 'top 82%' },
    })

    gsap.to(mediaWrap, {
      yPercent: -8,
      ease: 'none',
      scrollTrigger: { trigger: section, start: 'top bottom', end: 'bottom top', scrub: true },
    })

    gsap.to(copy, {
      yPercent: -4,
      ease: 'none',
      scrollTrigger: { trigger: section, start: 'top bottom', end: 'bottom top', scrub: true },
    })

    if (secondary) {
      gsap.to(secondary, {
        yPercent: 18,
        xPercent: section.classList.contains('align-right') ? -8 : 8,
        rotate: section.classList.contains('align-right') ? -6 : 6,
        ease: 'none',
        scrollTrigger: { trigger: section, start: 'top bottom', end: 'bottom top', scrub: true },
      })
    }

    gsap.from(chips, {
      y: 24,
      opacity: 0,
      duration: 0.55,
      stagger: 0.08,
      ease: 'power2.out',
      scrollTrigger: { trigger: section, start: 'top 68%' },
    })
  })
}

const syncLockedUiState = () => {
  const shouldLock = isMobileMenuOpen.value || Boolean(selectedImage.value)
  if (lenis) {
    if (shouldLock) lenis.stop()
    else lenis.start()
  }
  document.body.style.overflow = shouldLock ? 'hidden' : ''
}

onMounted(async () => {
  await nextTick()
  try {
    const savedTheme = window.localStorage.getItem(THEME_STORAGE_KEY)
    if (savedTheme) isDarkMode.value = savedTheme === 'dark'
    else isDarkMode.value = window.matchMedia('(prefers-color-scheme: dark)').matches
  } catch {
    isDarkMode.value = false
  }
  applyTheme()
  preloadBrandAssets(activeBrand.value, 'high')
  scheduleBrandWarmup(activeBrand.value.id)
  initLenis()
  initAmbientPointer()
  initStoryMediaObserver()
  initAnimations()
  keydownHandler = (event) => {
    if (isViewSourceShortcut(event)) {
      event.preventDefault()
      event.stopPropagation()
      return
    }

    if (event.key === 'Escape') {
      if (selectedImage.value) {
        closeImageViewer()
        return
      }
      if (galleryOpen.value) {
        closeGallery()
        return
      }
      if (isMobileMenuOpen.value) closeMobileMenu()
    }
    // Arrow key navigation - chỉ khi có gallery context
    if (selectedImage.value && currentGalleryImages.value.length > 1 && currentGalleryBrand.value) {
      if (event.key === 'ArrowLeft' && currentImageIndex.value > 0) {
        navigateImage(-1)
      } else if (event.key === 'ArrowRight' && currentImageIndex.value < currentGalleryImages.value.length - 1) {
        navigateImage(1)
      }
    }
  }
  resizeHandler = () => {
    if (!isMobileViewport()) closeMobileMenu()
    ScrollTrigger.refresh()
  }
  window.addEventListener('keydown', keydownHandler, true)
  window.addEventListener('resize', resizeHandler)
  setTimeout(() => ScrollTrigger.refresh(), 250)
})

watch(activeBrand, (brand) => {
  preloadBrandAssets(brand, 'high')
  scheduleBrandWarmup(brand.id)
})

watch(isDarkMode, (dark) => {
  applyTheme()
  try {
    window.localStorage.setItem(THEME_STORAGE_KEY, dark ? 'dark' : 'light')
  } catch {
    // Ignore storage limitations and keep the UI responsive.
  }
})

watch([isMobileMenuOpen, selectedImage], syncLockedUiState)

onUnmounted(() => {
  main.value?.removeEventListener('pointermove', pointerHandler)
  window.removeEventListener('keydown', keydownHandler, true)
  window.removeEventListener('resize', resizeHandler)
  storyMediaObserver?.disconnect()
  clearBrandWarmup()
  ScrollTrigger.getAll().forEach((trigger) => trigger.kill())
  if (lenis && lenisScrollHandler) lenis.off('scroll', lenisScrollHandler)
  if (gsapTicker) gsap.ticker.remove(gsapTicker)
  document.body.style.overflow = ''
  lenis?.destroy()
})
</script>
