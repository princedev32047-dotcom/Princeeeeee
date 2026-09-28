# Princeeee
/**
 * ============================================================================
 * ALEX MORGAN - FUTURISTIC PORTFOLIO SCRIPT
 * Vanilla JavaScript (ES6+)
 * No external frameworks or npm packages required.
 * Run directly by opening index.html in any modern web browser.
 * ============================================================================
 */

document.addEventListener('DOMContentLoaded', () => {
  'use strict';

  // --------------------------------------------------------------------------
  // 1. GLOBAL ENVIRONMENT & ACCESSIBILITY CHECK
  // --------------------------------------------------------------------------
  const isTouchDevice = () => {
    return (
      'ontouchstart' in window ||
      navigator.maxTouchPoints > 0 ||
      window.matchMedia('(pointer: coarse)').matches
    );
  };

  const prefersReducedMotion = window.matchMedia('(prefers-reduced-motion: reduce)').matches;

  // --------------------------------------------------------------------------
  // 2. CUSTOM CURSOR WITH SMOOTH LERP PHYSICS (Desktop Only)
  // --------------------------------------------------------------------------
  const cursorDot = document.getElementById('cursorDot');
  const cursorRing = document.getElementById('cursorRing');
  const mouseGlow = document.getElementById('mouseGlow');

  if (!isTouchDevice() && cursorDot && cursorRing) {
    let mouseX = window.innerWidth / 2;
    let mouseY = window.innerHeight / 2;
    let ringX = mouseX;
    let ringY = mouseY;
    let dotX = mouseX;
    let dotY = mouseY;
    let cursorVisible = false;

    // Track mouse coordinate movement
    window.addEventListener('mousemove', (e) => {
      mouseX = e.clientX;
      mouseY = e.clientY;

      if (!cursorVisible) {
        cursorDot.style.opacity = '1';
        cursorRing.style.opacity = '1';
        if (mouseGlow) mouseGlow.style.opacity = '1';
        cursorVisible = true;
      }

      // Update CSS variables for mouse-following gradient/spotlight
      document.documentElement.style.setProperty('--mouse-x', `${mouseX}px`);
      document.documentElement.style.setProperty('--mouse-y', `${mouseY}px`);
    });

    // Hide custom cursor when mouse leaves the viewport
    document.addEventListener('mouseleave', () => {
      cursorDot.style.opacity = '0';
      cursorRing.style.opacity = '0';
      if (mouseGlow) mouseGlow.style.opacity = '0';
      cursorVisible = false;
    });

    // Smooth animation loop for the outer trailing ring and glow
    const renderCursor = () => {
      // Linear interpolation (lerp) for smooth trailing delay
      ringX += (mouseX - ringX) * 0.16;
      ringY += (mouseY - ringY) * 0.16;
      dotX += (mouseX - dotX) * 0.75;
      dotY += (mouseY - dotY) * 0.75;

      cursorDot.style.transform = `translate(${dotX}px, ${dotY}px) translate(-50%, -50%)`;
      cursorRing.style.transform = `translate(${ringX}px, ${ringY}px) translate(-50%, -50%)`;

      if (mouseGlow) {
        mouseGlow.style.transform = `translate(${ringX}px, ${ringY}px) translate(-50%, -50%)`;
      }

      requestAnimationFrame(renderCursor);
    };

    if (!prefersReducedMotion) {
      requestAnimationFrame(renderCursor);
    }

    // Add cursor hover state to interactive targets
    const interactiveElements = document.querySelectorAll(
      'a, button, input, textarea, .glass-panel, .tech-chip, .social-pill, .tag-pill'
    );

    interactiveElements.forEach((el) => {
      el.addEventListener('mouseenter', () => document.body.classList.add('cursor-hover'));
      el.addEventListener('mouseleave', () => document.body.classList.remove('cursor-hover'));
    });
  }

  // --------------------------------------------------------------------------
  // 3. INTERACTIVE CONSTELLATION BACKGROUND CANVAS
  // --------------------------------------------------------------------------
  const canvas = document.getElementById('bg-canvas');
  if (canvas && !prefersReducedMotion) {
    const ctx = canvas.getContext('2d');
    let width = (canvas.width = window.innerWidth);
    let height = (canvas.height = window.innerHeight);

    // Responsive particle count based on display width
    let particleCount = Math.floor((width * height) / 22000);
    if (particleCount > 80) particleCount = 80;
    if (particleCount < 30) particleCount = 30;

    const particles = [];
    let mouse = { x: null, y: null, radius: 140 };

    window.addEventListener('mousemove', (e) => {
      mouse.x = e.clientX;
      mouse.y = e.clientY;
    });

    window.addEventListener('mouseleave', () => {
      mouse.x = null;
      mouse.y = null;
    });

    // Particle constructor
    class Particle {
      constructor() {
        this.x = Math.random() * width;
        this.y = Math.random() * height;
        this.vx = (Math.random() - 0.5) * 0.45;
        this.vy = (Math.random() - 0.5) * 0.45;
        this.radius = Math.random() * 1.5 + 0.8;
        this.baseColor = Math.random() > 0.5 ? 'rgba(0, 242, 254,' : 'rgba(157, 78, 221,';
        this.alpha = Math.random() * 0.4 + 0.2;
      }

      update() {
        this.x += this.vx;
        this.y += this.vy;

        // Wrap around boundaries
        if (this.x < 0) this.x = width;
        if (this.x > width) this.x = 0;
        if (this.y < 0) this.y = height;
        if (this.y > height) this.y = 0;

        // Subtle gentle mouse repulsion
        if (mouse.x !== null && mouse.y !== null) {
          const dx = mouse.x - this.x;
          const dy = mouse.y - this.y;
          const distance = Math.sqrt(dx * dx + dy * dy);

          if (distance < mouse.radius) {
            const force = (mouse.radius - distance) / mouse.radius;
            this.x -= (dx / distance) * force * 1.5;
            this.y -= (dy / distance) * force * 1.5;
          }
        }
      }

      draw() {
        ctx.beginPath();
        ctx.arc(this.x, this.y, this.radius, 0, Math.PI * 2);
        ctx.fillStyle = `${this.baseColor} ${this.alpha})`;
        ctx.fill();
      }
    }

    // Initialize particles
    for (let i = 0; i < particleCount; i++) {
      particles.push(new Particle());
    }

    // Canvas render loop
    const animateCanvas = () => {
      ctx.clearRect(0, 0, width, height);

      // Connect particles within proximity
      for (let i = 0; i < particles.length; i++) {
        particles[i].update();
        particles[i].draw();

        for (let j = i + 1; j < particles.length; j++) {
          const dx = particles[i].x - particles[j].x;
          const dy = particles[i].y - particles[j].y;
          const dist = Math.sqrt(dx * dx + dy * dy);

          if (dist < 110) {
            const lineAlpha = (1 - dist / 110) * 0.15;
            ctx.beginPath();
            ctx.moveTo(particles[i].x, particles[i].y);
            ctx.lineTo(particles[j].x, particles[j].y);
            ctx.strokeStyle = `rgba(0, 242, 254, ${lineAlpha})`;
            ctx.lineWidth = 0.75;
            ctx.stroke();
          }
        }
      }

      requestAnimationFrame(animateCanvas);
    };

    requestAnimationFrame(animateCanvas);

    // Debounced window resize handler for canvas
    let resizeTimer;
    window.addEventListener('resize', () => {
      clearTimeout(resizeTimer);
      resizeTimer = setTimeout(() => {
        width = canvas.width = window.innerWidth;
        height = canvas.height = window.innerHeight;
      }, 150);
    });
  }

  // --------------------------------------------------------------------------
  // 4. STICKY BLURRED NAVBAR & SCROLL SPY
  // --------------------------------------------------------------------------
  const navbar = document.getElementById('navbar');
  const navLinks = document.querySelectorAll('.nav-link');
  const sections = document.querySelectorAll('section[id]');
  const backToTopBtn = document.getElementById('backToTopBtn');

  const handleScroll = () => {
    const scrollPos = window.scrollY;

    // Toggle navbar frosted glass background
    if (scrollPos > 40) {
      navbar.classList.add('scrolled');
    } else {
      navbar.classList.remove('scrolled');
    }

    // Toggle Back to Top button visibility
    if (backToTopBtn) {
      if (scrollPos > 400) {
        backToTopBtn.classList.add('visible');
      } else {
        backToTopBtn.classList.remove('visible');
      }
    }

    // Scroll spy: highlight the active nav item
    sections.forEach((section) => {
      const top = section.offsetTop - 120;
      const height = section.offsetHeight;
      const id = section.getAttribute('id');

      if (scrollPos >= top && scrollPos < top + height) {
        navLinks.forEach((link) => {
          link.classList.remove('active');
          if (link.getAttribute('href') === `#${id}`) {
            link.classList.add('active');
          }
        });
      }
    });
  };

  window.addEventListener('scroll', handleScroll, { passive: true });
  handleScroll(); // Trigger initial check

  // Smooth scroll to top when button clicked
  if (backToTopBtn) {
    backToTopBtn.addEventListener('click', () => {
      window.scrollTo({
        top: 0,
        behavior: 'smooth',
      });
    });
  }

  // --------------------------------------------------------------------------
  // 5. MOBILE HAMBURGER MENU & DRAWER
  // --------------------------------------------------------------------------
  const hamburgerBtn = document.getElementById('hamburgerBtn');
  const mobileDrawer = document.getElementById('mobileDrawer');
  const drawerCloseBtn = document.getElementById('drawerCloseBtn');
  const drawerBackdrop = document.getElementById('drawerBackdrop');
  const mobileNavLinks = document.querySelectorAll('.mobile-nav-link, .mobile-cta-btn');

  const toggleMobileMenu = (forceState) => {
    const isOpen = typeof forceState === 'boolean' ? forceState : !mobileDrawer.classList.contains('open');

    if (isOpen) {
      mobileDrawer.classList.add('open');
      if (drawerBackdrop) drawerBackdrop.classList.add('active');
      hamburgerBtn.classList.add('active');
      hamburgerBtn.setAttribute('aria-expanded', 'true');
      mobileDrawer.setAttribute('aria-hidden', 'false');
      document.body.style.overflow = 'hidden'; // Lock background scroll
    } else {
      mobileDrawer.classList.remove('open');
      if (drawerBackdrop) drawerBackdrop.classList.remove('active');
      hamburgerBtn.classList.remove('active');
      hamburgerBtn.setAttribute('aria-expanded', 'false');
      mobileDrawer.setAttribute('aria-hidden', 'true');
      document.body.style.overflow = '';
    }
  };

  if (hamburgerBtn && mobileDrawer) {
    hamburgerBtn.addEventListener('click', () => toggleMobileMenu());

    if (drawerCloseBtn) {
      drawerCloseBtn.addEventListener('click', () => toggleMobileMenu(false));
    }

    if (drawerBackdrop) {
      drawerBackdrop.addEventListener('click', () => toggleMobileMenu(false));
    }

    // Close menu when clicking any mobile navigation link
    mobileNavLinks.forEach((link) => {
      link.addEventListener('click', () => toggleMobileMenu(false));
    });

    // Close drawer if user presses Escape key
    document.addEventListener('keydown', (e) => {
      if (e.key === 'Escape' && mobileDrawer.classList.contains('open')) {
        toggleMobileMenu(false);
      }
    });
  }

  // --------------------------------------------------------------------------
  // 6. SCROLL REVEAL ANIMATIONS (IntersectionObserver)
  // --------------------------------------------------------------------------
  const revealElements = document.querySelectorAll(
    '.reveal-fade, .reveal-slide-up, .reveal-slide-left, .reveal-slide-right, .reveal-scale'
  );

  if ('IntersectionObserver' in window && !prefersReducedMotion) {
    const revealObserver = new IntersectionObserver(
      (entries, observer) => {
        entries.forEach((entry) => {
          if (entry.isIntersecting) {
            entry.target.classList.add('in-view');
            // Unobserve after revealing to save performance
            observer.unobserve(entry.target);
          }
        });
      },
      {
        root: null,
        threshold: 0.12,
        rootMargin: '0px 0px -40px 0px',
      }
    );

    revealElements.forEach((el) => revealObserver.observe(el));
  } else {
    // Fallback: make all elements immediately visible if observer not supported
    revealElements.forEach((el) => el.classList.add('in-view'));
  }

  // --------------------------------------------------------------------------
  // 7. ANIMATED STATISTIC COUNTERS
  // --------------------------------------------------------------------------
  const counters = document.querySelectorAll('.counter');
  let countersAnimated = false;

  const animateCounters = () => {
    if (countersAnimated) return;

    counters.forEach((counter) => {
      const target = +counter.getAttribute('data-target');
      const duration = 1800; // milliseconds
      const frameDuration = 1000 / 60;
      const totalFrames = Math.round(duration / frameDuration);
      let frame = 0;

      const easeOutQuart = (x) => 1 - Math.pow(1 - x, 4);

      const updateCounter = () => {
        frame++;
        const progress = easeOutQuart(frame / totalFrames);
        const currentCount = Math.round(target * progress);

        counter.textContent = currentCount;

        if (frame < totalFrames) {
          requestAnimationFrame(updateCounter);
        } else {
          counter.textContent = target;
        }
      };

      requestAnimationFrame(updateCounter);
    });

    countersAnimated = true;
  };

  const statsSection = document.querySelector('.about-stats-column');
  if (statsSection && 'IntersectionObserver' in window && !prefersReducedMotion) {
    const statsObserver = new IntersectionObserver(
      (entries, observer) => {
        entries.forEach((entry) => {
          if (entry.isIntersecting) {
            animateCounters();
            observer.unobserve(entry.target);
          }
        });
      },
      { threshold: 0.3 }
    );

    statsObserver.observe(statsSection);
  } else {
    // Fallback if reduced motion or no IntersectionObserver
    counters.forEach((c) => (c.textContent = c.getAttribute('data-target')));
  }

  // --------------------------------------------------------------------------
  // 8. 3D TILT EFFECT ON PROJECT CARDS (Vanilla JS)
  // --------------------------------------------------------------------------
  const tiltCards = document.querySelectorAll('[data-tilt]');

  if (!isTouchDevice() && !prefersReducedMotion) {
    tiltCards.forEach((card) => {
      const glare = card.querySelector('.card-glare');

      card.addEventListener('mousemove', (e) => {
        const rect = card.getBoundingClientRect();
        const x = e.clientX - rect.left;
        const y = e.clientY - rect.top;

        // Calculate normalized offset from center (-1 to 1)
        const centerX = rect.width / 2;
        const centerY = rect.height / 2;
        const rotateX = ((y - centerY) / centerY) * -8; // Max 8 deg
        const rotateY = ((x - centerX) / centerX) * 8;

        card.style.transform = `perspective(1000px) rotateX(${rotateX.toFixed(2)}deg) rotateY(${rotateY.toFixed(2)}deg) translateY(-6px)`;

        // Specular highlight glare position
        if (glare) {
          const glareX = (x / rect.width) * 100;
          const glareY = (y / rect.height) * 100;
          glare.style.background = `radial-gradient(circle at ${glareX}% ${glareY}%, rgba(255, 255, 255, 0.18), transparent 60%)`;
        }
      });

      card.addEventListener('mouseleave', () => {
        card.style.transform = 'perspective(1000px) rotateX(0deg) rotateY(0deg) translateY(0px)';
      });
    });
  }

  // --------------------------------------------------------------------------
  // 9. DYNAMIC TYPING EFFECT FOR HERO SUBTITLE
  // --------------------------------------------------------------------------
  const typingElement = document.getElementById('typingText');
  if (typingElement && !prefersReducedMotion) {
    const roles = [
      'Computer Science Student & Developer',
      'Full-Stack Web Enthusiast',
      'Algorithmic Problem Solver',
      'Open-Source Contributor',
    ];

    let roleIndex = 0;
    let charIndex = roles[0].length;
    let isDeleting = false;
    let typingSpeed = 70;

    const typeRole = () => {
      const currentRole = roles[roleIndex];

      if (isDeleting) {
        typingElement.textContent = currentRole.substring(0, charIndex - 1);
        charIndex--;
        typingSpeed = 35;
      } else {
        typingElement.textContent = currentRole.substring(0, charIndex + 1);
        charIndex++;
        typingSpeed = 75;
      }

      // Check boundary conditions
      if (!isDeleting && charIndex === currentRole.length) {
        // Pause at complete text
        typingSpeed = 2200;
        isDeleting = true;
      } else if (isDeleting && charIndex === 0) {
        isDeleting = false;
        roleIndex = (roleIndex + 1) % roles.length;
        typingSpeed = 400; // Brief pause before starting next word
      }

      setTimeout(typeRole, typingSpeed);
    };

    // Kick off typing sequence after initial hero render
    setTimeout(typeRole, 2000);
  }

  // --------------------------------------------------------------------------
  // 10. CONTACT FORM VALIDATION & ANIMATED SUBMISSION
  // --------------------------------------------------------------------------
  const contactForm = document.getElementById('contactForm');
  const submitBtn = document.getElementById('submitBtn');
  const formSuccessAlert = document.getElementById('formSuccessAlert');
  const closeAlertBtn = document.getElementById('closeAlertBtn');

  if (contactForm && submitBtn) {
    const nameInput = document.getElementById('userName');
    const emailInput = document.getElementById('userEmail');
    const messageInput = document.getElementById('userMessage');

    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;

    // Helper to validate individual form fields
    const validateField = (input, isValid) => {
      const group = input.closest('.form-group');
      if (group) {
        if (!isValid) {
          group.classList.add('has-error');
        } else {
          group.classList.remove('has-error');
        }
      }
      return isValid;
    };

    // Real-time input clearing of errors on user typing
    [nameInput, emailInput, messageInput].forEach((input) => {
      if (input) {
        input.addEventListener('input', () => {
          const group = input.closest('.form-group');
          if (group && group.classList.contains('has-error')) {
            group.classList.remove('has-error');
          }
        });
      }
    });

    contactForm.addEventListener('submit', (e) => {
      e.preventDefault();

      // Form validation checks
      const isNameValid = validateField(nameInput, nameInput.value.trim().length > 0);
      const isEmailValid = validateField(emailInput, emailRegex.test(emailInput.value.trim()));
      const isMessageValid = validateField(messageInput, messageInput.value.trim().length > 0);

      if (!isNameValid || !isEmailValid || !isMessageValid) {
        return;
      }

      // Enter simulated submission state
      submitBtn.classList.add('loading');
      submitBtn.disabled = true;

      /**
       * BACKEND INTEGRATION NOTE:
       * In a production environment, send data using fetch() to:
       * - Formspree (https://formspree.io/f/your_form_id)
       * - EmailJS (https://www.emailjs.com/)
       * - A serverless API endpoint or Express/Node.js backend.
       *
       * Example:
       * fetch('https://formspree.io/f/your_id', {
       *   method: 'POST',
       *   headers: { 'Content-Type': 'application/json' },
       *   body: JSON.stringify({ name: nameInput.value, email: emailInput.value, message: messageInput.value })
       * });
       */

      // Simulated network latency (1.1 seconds)
      setTimeout(() => {
        submitBtn.classList.remove('loading');
        submitBtn.disabled = false;

        // Reset inputs
        contactForm.reset();

        // Reveal animated success notification
        if (formSuccessAlert) {
          formSuccessAlert.classList.add('show');

          // Auto scroll alert into view if needed
          formSuccessAlert.scrollIntoView({ behavior: 'smooth', block: 'nearest' });

          // Auto-hide alert after 7 seconds
          setTimeout(() => {
            formSuccessAlert.classList.remove('show');
          }, 7000);
        }
      }, 1100);
    });

    // Close alert button handler
    if (closeAlertBtn && formSuccessAlert) {
      closeAlertBtn.addEventListener('click', () => {
        formSuccessAlert.classList.remove('show');
      });
    }
  }

  // --------------------------------------------------------------------------
  // 11. DYNAMIC CURRENT YEAR FOR FOOTER
  // --------------------------------------------------------------------------
  const currentYearSpan = document.getElementById('currentYear');
  if (currentYearSpan) {
    currentYearSpan.textContent = new Date().getFullYear();
  }

  // --------------------------------------------------------------------------
  // 12. PARALLAX EFFECT FOR HERO ORBITAL VISUAL ON MOUSE MOVE
  // --------------------------------------------------------------------------
  const heroVisual = document.getElementById('heroVisual');
  if (heroVisual && !isTouchDevice() && !prefersReducedMotion) {
    window.addEventListener('mousemove', (e) => {
      const centerX = window.innerWidth / 2;
      const centerY = window.innerHeight / 2;
      const moveX = (e.clientX - centerX) * 0.025;
      const moveY = (e.clientY - centerY) * 0.025;

      heroVisual.style.transform = `translate(${moveX}px, ${moveY}px)`;
    });
  }

  console.log(
    '%c[Portfolio Initialized]%c Futuristic developer portfolio running smoothly with Vanilla HTML/CSS/JS.',
    'color: #00f2fe; font-weight: bold; font-size: 12px;',
    'color: #a855f7; font-size: 12px;'
  );
});
