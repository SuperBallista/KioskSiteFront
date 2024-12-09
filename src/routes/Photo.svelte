<script>
    import { onMount } from 'svelte';
  
    $: photos = [];
    let id = '';
  
    onMount(async () => {
      // 현재 URL을 파싱해서 `id` 값 추출
      const urlParams = new URLSearchParams(window.location.search);
      id = urlParams.get('id');
  
      // 서버에서 사진 데이터 로드
      const response = await fetch(`/photo/load?id=${id}`);
      photos = await response.json();
      
    });
  
    let modalImage = '';
    let modalTitle = '';
    let isModalOpen = false;
  
    function openModal(imageUrl, title) {
      modalImage = imageUrl;
      modalTitle = title;
      isModalOpen = true;
    }
  
    function closeModal() {
      isModalOpen = false;
    }
  
    function scrollPage(offset) {
      window.scrollBy({
        top: offset,
        behavior: 'smooth'
      });
    }
  </script>
  
  <style>
  
  .gallery {
      display: flex;
      flex-wrap: wrap;
      justify-content: space-evenly;
      width: 100%;
      max-width: 1000px;
      margin: 0 auto;
    }
  
    .photo {
      position: relative;
      width: 300px;
      height: 300px;
      border-radius: 12px;
      overflow: hidden;
      cursor: pointer;
      transition: transform 0.3s ease;
      box-shadow: 0 6px 12px rgba(0, 0, 0, 0.1);
    }
  
    .photo img {
      width: 100%;
      height: 100%;
      object-fit: cover;
      transition: transform 0.3s ease;
    }
  
    .photo:hover img {
      transform: scale(1.08);
    }
  
    .photo-title {
      position: absolute;
      bottom: 0;
      left: 0;
      right: 0;
      background-color: rgba(0, 0, 0, 0.6);
      color: white;
      text-align: center;
      padding: 10px;
    }
  
    .modal-overlay {
      position: fixed;
      top: 0;
      left: 0;
      right: 0;
      bottom: 0;
      display: flex;
      align-items: center;
      justify-content: center;
      background-color: rgba(0, 0, 0, 0.8);
      z-index: 1000;
      visibility: hidden;
      opacity: 0;
      transition: opacity 0.3s ease, visibility 0.3s ease;
    }
  
    .modal-overlay.open {
      visibility: visible;
      opacity: 1;
    }
  
    .modal-content {
      background: black;
      padding: 20px;
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      max-height: 90%;
      overflow: hidden;
      position: relative;
      border-radius: 12px;
    }
  
    .modal-content img {
      width: 100%;
      height: auto;
      max-height: 80vh;
      object-fit: contain;
    }
  
    .modal-title {
      color: white;
      font-size: 1.5em;
      margin-top: 10px;
      text-align: center;
    }
  
    .close-button {
      position: absolute;
      top: 10px;
      right: 10px;
      background-color: #333;
      color: white;
      padding: 10px;
      border-radius: 8px;
      cursor: pointer;
      font-size: 1em;
    }
  
    .scroll-buttons {
      position: fixed;
      right: 10px;
      top: 50%;
      transform: translateY(-50%);
      display: flex;
      flex-direction: column;
      gap: 10px;
    }
  
    .scroll-button {
      background-color: #333;
      color: white;
      padding: 10px 15px;
      border-radius: 8px;
      cursor: pointer;
      font-size: 1.2em;
      text-align: center;
      user-select: none;
    }
  </style>
  
  <div class="gallery">
    {#each photos as photo}
      <button 
        class="photo" 
        on:click={() => openModal(photo.src, photo.title)}
        on:keydown={(e) => e.key === 'Enter' && openModal(photo.src, photo.title)}
      >
        <img src={photo.src} alt={photo.title} />
        <div class="photo-title">{photo.title}</div>
      </button>
    {/each}
  </div>
  
  <!-- 모달 -->
  <button 
    class="modal-overlay {isModalOpen ? 'open' : ''}" 
    on:click={() => closeModal()}
    on:keydown={(e) => e.key === 'Escape' && closeModal()}
  >
    <div 
      class="modal-content" 
      on:click|stopPropagation 
      on:keydown|stopPropagation={() => {}}
    >
      <img src={modalImage} alt={modalTitle} />
      <div class="modal-title">{modalTitle}</div>
      <button 
        class="close-button" 
        on:click={() => closeModal()}
        on:keydown={(e) => e.key === 'Enter' && closeModal()}
      >닫기</button>
    </div>
  </button>
  
  <!-- 스크롤 버튼 -->
  <div class="scroll-buttons">
    <button 
      class="scroll-button" 
      on:click={() => scrollPage(-500)}
      on:keydown={(e) => e.key === 'Enter' && scrollPage(-500)}
    >▲</button>
    <button 
      class="scroll-button" 
      on:click={() => scrollPage(500)}
      on:keydown={(e) => e.key === 'Enter' && scrollPage(500)}
    >▼</button>
  </div>
  