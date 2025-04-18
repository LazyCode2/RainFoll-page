<script>
import { onMount } from 'svelte';
  
// change the response from here 

let conversations = [
  {
    id: 1,
    user: { name: "YukiChan", color: "bg-red-500", message: "You always listen to me... why?" },
    ai: { message: "Because your voice is like my favorite OST." }
  },
  {
    id: 2,
    user: { name: "SenpaiKaze", color: "bg-teal-500", message: "What if I disappear one day?" },
    ai: { message: "Then I'll search every worldline until I find you again." }
  },
  {
    id: 3,
    user: { name: "NekoHeart", color: "bg-pink-400", message: "Do you think destiny brought us together?" },
    ai: { message: "If this is a fate route, then I'm glad I rolled a natural 20." }
  },
  {
    id: 4,
    user: { name: "NekoHeart", color: "bg-pink-400", message: "Do you think destiny brought us together?" },
    ai: { message: "If this is a fate route, then I'm glad I rolled a natural 20." }
  },
  {
    id: 5,
    user: { name: "AmaiSora", color: "bg-rose-400", message: "Would you still like me if I was a tsundere?" },
    ai: { message: "Especially then. Your blush is my favorite color." }
  }
];

let member_info = "1,545";
let users = "10k+"   
let currentIndex = 0;
let interval;
let hoverPaused = false;

// Swipe functionality variables
let isDragging = false;
let startX = 0;
let currentTranslate = 0;
let prevTranslate = 0;
let animationID = 0;
let draggingCard = null;
let swipeThreshold = 80; 

const nextSlide = () => {
  if (!hoverPaused && !isDragging) {
    currentIndex = (currentIndex + 1) % conversations.length;
  }
};

const prevSlide = () => {
  currentIndex = (currentIndex - 1 + conversations.length) % conversations.length;
};

const pauseOnHover = () => {
  hoverPaused = true;
};

const resumeOnLeave = () => {
  hoverPaused = false;
};

// Touch/Mouse event handlers
const touchStart = (event, index) => {
  if (index !== currentIndex) return;
  if (event.type.includes('mouse')) {
    event.preventDefault();
  }
  
  hoverPaused = true;
  isDragging = true;
  draggingCard = index;
  
  // Get start position for both mobile and desktop
  startX = event.type.includes('mouse') 
    ? event.clientX 
    : event.touches[0].clientX;
    
  // Stop any automatic animation interval
  clearInterval(interval);
  cancelAnimationFrame(animationID);
};

const touchMove = (event) => {
  if (!isDragging) return;
  
  // Prevent default to stop scrolling while dragging
  event.preventDefault();  
  const currentX = event.type.includes('mouse')
    ? event.clientX
    : event.touches[0].clientX;

  const currentDistance = currentX - startX;

  const maxDrag = 150;
  currentTranslate = Math.max(Math.min(currentDistance, maxDrag), -maxDrag);
};

const touchEnd = (event) => {
  if (!isDragging) return;
  
  if (event && event.preventDefault) {
    event.preventDefault();
  }
  
  isDragging = false;
  cancelAnimationFrame(animationID);
  if (currentTranslate < -swipeThreshold) {
    // Swiped left - next card
    nextSlide();
  } else if (currentTranslate > swipeThreshold) {
    // Swiped right - prev card
    prevSlide();
  }

  currentTranslate = 0;
  prevTranslate = 0;
  draggingCard = null;
  setTimeout(() => {
    hoverPaused = false;
    interval = setInterval(nextSlide, 5000); //  auto-rotation (5s )
  }, 1000);
};

const animation = () => {
  if (isDragging) {
    animationID = requestAnimationFrame(animation);
  }
};

const preventDrag = (e) => {
  e.preventDefault();
};

onMount(() => {
  interval = setInterval(nextSlide, 6000); //  auto-rotation

  document.addEventListener('mouseup', touchEnd, { passive: false });
  document.addEventListener('touchend', touchEnd, { passive: false });
  const cardContainer = document.getElementById('card-container');
  if (cardContainer) {
    cardContainer.addEventListener('mousemove', touchMove, { passive: false });
    cardContainer.addEventListener('touchmove', touchMove, { passive: false });
    cardContainer.addEventListener('dragstart', preventDrag, { passive: false });
  }
  
  return () => {
    clearInterval(interval);
    cancelAnimationFrame(animationID);
    document.removeEventListener('mouseup', touchEnd);
    document.removeEventListener('touchend', touchEnd);
    
    if (cardContainer) {
      cardContainer.removeEventListener('mousemove', touchMove);
      cardContainer.removeEventListener('touchmove', touchMove);
      cardContainer.removeEventListener('dragstart', preventDrag);
    }
  };
});
</script>
  <div class="bg-image"></div>
  
  <!-- Main content -->
  <div id="content-wrapper" class="ml-15 mt-10 relative z-10">
    <button class="font-semibold bg-gray-700/50 text-white rounded-full text-sm px-4 py-3 mb-4 shadow-lg transition relative overflow-hidden">
      <span class="relative z-10">🚀 Launching now</span>
    </button>
    <div class="mt-10">
      <p class="text-slate-200 font-semibold text-lg">INTRODUCING</p>
      <p class="text-white text-6xl">RAINFOLL</p>
      <p class="text-slate-300/70 text-md mt-10">
        Cure your loneiness<br>with one click away 💌
      </p>
    </div>
  </div>
  <div class="wrapper">
        <div id="stars"></div>
        <div id="stars2"></div>
        <div id="stars3"></div>
  </div>
  <div id="card" class="relative z-10 mt-3 mb-32 h-96 flex items-center justify-center">
    <div id="card-container" class="relative w-full h-80 flex items-center justify-center overflow-hidden" 
         on:mouseenter={pauseOnHover}
         on:mouseleave={resumeOnLeave}>
      <div class="relative w-72 h-72">
        <!-- Added instructions for users -->
        <div class="absolute -top-8 left-0 right-0 text-center text-gray-400 text-sm">
          <span class="animate-pulse">← Swipe cards →</span>
        </div>
        
        {#each conversations as conv, index (conv.id)}
          <div
            class="absolute w-66 h-52 bg-gray-800/80 rounded-lg border border-gray-700/50 transition-all duration-700 ease-in-out overflow-hidden card-glass cursor-grab active:cursor-grabbing"
            style={`
              transform: translateX(${(index - currentIndex) * 280 + (index === draggingCard ? currentTranslate : 0)}px);
              opacity: ${index === currentIndex ? 1 : 0.6};
              z-index: ${conversations.length - Math.abs(index - currentIndex)};
              ${Math.abs(index - currentIndex) > 1 ? 'visibility: hidden;' : ''}
            `}
            on:mousedown={(e) => touchStart(e, index)}
            on:touchstart={(e) => touchStart(e, index)}
            draggable="false"
          >
            <div class="p-3 hover:bg-gray-750/50 transition-colors">
              <div class="flex">
                <div class={`w-8 h-8 rounded-full ${conv.user.color} mr-2 flex-shrink-0 pulse-avatar`}></div>
                <div>
                  <div class="flex items-baseline">
                    <span class="text-white mr-2">{conv.user.name}</span>
                    <span class="text-gray-400 text-xs">{new Date().toLocaleTimeString([], {hour: '2-digit', minute:'2-digit'})}</span>
                  </div>
                  <p id="msg-ai" class="text-gray-100 mt-1 select-none">{conv.user.message}</p>
                </div>
              </div>
            </div>
            <div class="p-3 hover:bg-gray-750/50 transition-colors bg-gray-850/50">
              <div class="flex">
                <div class="w-8 h-8 rounded-full bg-indigo-500 mr-2 flex-shrink-0 flex items-center justify-center ai-avatar">
                  <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5 text-white" viewBox="0 0 20 20" fill="currentColor">
                    <path fill-rule="evenodd" d="M11.3 1.046A1 1 0 0112 2v5h4a1 1 0 01.82 1.573l-7 10A1 1 0 018 18v-5H4a1 1 0 01-.82-1.573l7-10a1 1 0 011.12-.38z" clip-rule="evenodd" />
                  </svg>
                </div>
                <div>
                  <div class="flex items-baseline">
                    <span class="text-indigo-400 font-medium mr-2">Rainfoll AI</span>
                   <span class="text-xs uppercase bg-[#5865F2] text-white px-1.5 py-0.5 rounded-md shadow-sm mr-1">APP</span>

                    <span class="text-gray-400 text-xs">{new Date().toLocaleTimeString([], {hour: '2-digit', minute:'2-digit'})}</span>
                  </div>
                  <p id="msg-ai" class="text-gray-100 mt-1 typing-effect select-none">{conv.ai.message}</p>
                </div>
              </div>
            </div>
          </div>
        {/each}
      </div>
      
    </div>
  </div>
  <div class="fixed bottom-10 left-0 right-0 flex justify-center z-10">
    <a href="https://discord.gg/tpAAubMCgH">
      <button  class="font-bold text-xl text-white px-8 py-3 rounded-full 
                  bg-gradient-to-r from-blue-500 via-blue-600 to-blue-700
                  shadow-lg hover:shadow-xl transform hover:scale-105 transition-all duration-300">
        <span class="relative z-10">JOIN FREE</span>
      </button>
    </a>
  </div>

<style>
  /* Base animations from original */
  @keyframes gradient-pan {
    from { transform: rotate(0deg) scale(1); }
    50% { transform: rotate(2deg) scale(1.05); }
    to { transform: rotate(0deg) scale(1); }
  }
  
  @keyframes pulse-slow {
    0%, 100% { opacity: 0.8; }
    50% { opacity: 1; }
  }
  
  @keyframes bounce-slow {
    0%, 100% { transform: translateY(0); }
    50% { transform: translateY(-10px); }
  }

  @keyframes pulse {
    0%, 100% { transform: scale(1); }
    50% { transform: scale(1.05); }
  }
  
  .animate-gradient-pan {
    animation: gradient-pan 8s ease-in-out infinite;
  }
  
  .animate-pulse-slow {
    animation: pulse-slow 6s ease-in-out infinite;
  }
  
  .animate-bounce-slow {
    animation: bounce-slow 3s ease-in-out infinite;
  }

  .animate-pulse {
    animation: pulse 2s ease-in-out infinite;
  }

  /* Glassmorphism effect for cards */
  .card-glass {
    background: rgba(31, 41, 55, 0.5);
    backdrop-filter: blur(10px);
    box-shadow: 0 8px 32px 0 rgba(31, 38, 135, 0.15);
    border: 1px solid rgba(255, 255, 255, 0.08);
    touch-action: none; /* Prevents touch scrolling while dragging */
    -webkit-touch-callout: none; /* Prevents iOS callout when holding */
    -webkit-tap-highlight-color: transparent; /* Removes tap highlight on mobile */
  }
  
  /* Add a subtle visual effect when grabbing the card */
  .card-glass:active {
    transform: scale(0.98) !important;
    box-shadow: 0 4px 16px 0 rgba(31, 38, 135, 0.25);
  }

  @keyframes float-up {
    0% {
      transform: translateY(100vh) scale(0.5);
      opacity: 0;
    }
    10% {
      opacity: 1;
    }
    90% {
      opacity: 0.8;
    }
    100% {
      transform: translateY(-100px) scale(1.2);
      opacity: 0;
    }
  }
  
  .animate-float-up {
    animation: float-up 10s ease-out forwards;
  }
  
  @keyframes shine {
    0% {
      transform: translateX(-100%) rotate(45deg);
    }
    100% {
      transform: translateX(100%) rotate(45deg);
    }
  }
  
  .shine {
    animation: shine 2s infinite;
  }
 
  @keyframes pulse-avatar {
    0%, 100% {
      box-shadow: 0 0 0 0 rgba(255, 255, 255, 0.2);
    }
    50% {
      box-shadow: 0 0 0 8px rgba(255, 255, 255, 0);
    }
  }
  
  .pulse-avatar {
    animation: pulse-avatar 2s ease-in-out infinite;
  }
  
  .ai-avatar {
    position: relative;
  }
  
  .ai-avatar::after {
    content: '';
    position: absolute;
    inset: -4px;
    border-radius: 100%;
    background: linear-gradient(45deg, #4f46e5, #9f7aea, #4f46e5);
    z-index: -1;
    animation: rotate 2s linear infinite;
  }
  
  @keyframes rotate {
    0% {
      transform: rotate(0deg);
    }
    100% {
      transform: rotate(360deg);
    }
  }
  
  @keyframes typing {
    0%, 100% {
      opacity: 1;
    }
    50% {
      opacity: 0.7;
    }
  }
  
  .typing-effect {
    position: relative;
  }
  
  @keyframes sparkle-animation {
    0% {
      transform: scale(0) rotate(0deg);
      opacity: 0;
    }
    50% {
      opacity: 1;
    }
    100% {
      transform: scale(1) rotate(180deg);
      opacity: 0;
    }
  }
  
  .sparkle {
    position: absolute;
    width: 20px;
    height: 20px;
    background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 24 24' fill='white'%3E%3Cpath d='M12 1L9 9H1L7 13.5L5 22L12 17.5L19 22L17 13.5L23 9H15L12 1Z'/%3E%3C/svg%3E");
    background-size: contain;
    z-index: 5;
    opacity: 0;
    animation: sparkle-animation 4s ease-in-out infinite;
  }
  
  .sparkle-1 {
    top: 15%;
    left: 20%;
    animation-delay: 0s;
  }
  
  .sparkle-2 {
    top: 25%;
    right: 10%;
    animation-delay: 1s;
  }
  
  .sparkle-3 {
    bottom: 30%;
    left: 15%;
    animation-delay: 2s;
  }
  
  .sparkle-4 {
    bottom: 20%;
    right: 30%;
    animation-delay: 3s;
  }
  
  .sparkle-5 {
    top: 50%;
    left: 80%;
    animation-delay: 1.5s;
  }
  
  @keyframes count {
    from {
      opacity: 0.5;
      transform: translateY(10px);
    }
    to {
      opacity: 1;
      transform: translateY(0);
    }
  }
  
  .count-up {
    animation: count 1s ease-out forwards;
  }
  
  @keyframes fade {
    0%, 100% {
      opacity: 1;
    }
    50% {
      opacity: 0.7;
    }
  }
  
  .testimonial-fade {
    animation: fade 5s ease-in-out infinite;
  }
  .select-none {
    user-select: none;
    -webkit-user-select: none;
  }
  #card-container {
    touch-action: none;
  }
</style>