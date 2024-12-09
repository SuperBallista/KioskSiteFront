<script>
    import { onMount } from 'svelte';
    import { jwtDecode } from 'jwt-decode';
    import { Link } from 'svelte-routing';
  
    $: photos = [];
    let userId = '';
    let isLoading = false; // 로딩 상태
    
    // 세션 스토리지에서 JWT 토큰 디코딩 및 사용자 ID 추출
    function getIdFromToken() {
      const token = sessionStorage.getItem("token");
      if (!token) {
        console.error("No token found in session storage");
        return null;
      }
  
      try {
        const decoded = jwtDecode(token);
        console.log("Decoded token:", decoded);
        return decoded.id;
      } catch (error) {
        console.error("Error decoding token:", error);
        return null;
      }
    }
  
    // 컴포넌트가 로드될 때 서버에서 사진 목록 로드
    onMount(async () => {
      userId = getIdFromToken();
      if (!userId) return;
  
      try {
        const response = await fetch(`/photo/load?id=${userId}`);
        if (response.ok) {
          photos = await response.json();
        } else {
          console.error("Failed to load photos:", await response.text());
        }
      } catch (error) {
        console.error("Error loading photos:", error);
      }
    });
  
    // 사진 삭제 요청
    async function deletePhoto(photoId) {
      isLoading = true; // 로딩 상태 활성화

      try {
        const response = await fetch(`/photo/delete`, {
          method: 'POST', // 삭제 요청
          headers: {
            'Content-Type': 'application/json',
            Authorization: `Bearer ${sessionStorage.getItem('token')}`, // 토큰 추가
          },
          body: JSON.stringify({ id: photoId }), // 삭제할 사진의 ID 전달
        });
  
        if (response.ok) {
          photos = photos.filter(photo => photo.img_id !== photoId); // 성공적으로 삭제된 사진을 UI에서 제거
          console.log(`Photo ${photoId} deleted successfully`);
        } else {
          console.error(`Failed to delete photo ${photoId}:`, await response.text());
        }
      } catch (error) {
        console.error(`Error deleting photo ${photoId}:`, error);
      } finally {
        isLoading = false; // 로딩 상태 비활성화
      }
    }
  </script>
  
  <style>
    .gallery {
      display: grid;
      grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
      gap: 20px;
      max-width: 1000px;
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

    .link {
  background-color: #007bff;
  color: white;
  border: none;
  padding: 10px 20px;
  border-radius: 20px;
  font-size: 16px;
  font-weight: bold; /* 텍스트 강조 */
  text-align: center; /* 텍스트 가운데 정렬 */
  display: inline-block; /* 크기 조정 가능 */
  cursor: pointer;
  transition: background-color 0.3s ease, box-shadow 0.3s ease, transform 0.2s ease;
  text-decoration: none; /* 링크처럼 보이는 밑줄 제거 */
}

.link:hover {
  background-color: #0056b3;
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.1);
  transform: translateY(-2px); /* 호버 시 살짝 떠오르는 효과 */
}

.link:active {
  transform: translateY(2px); /* 클릭 시 눌리는 효과 */
  background-color: #004494; /* 클릭 시 색상 변화 */
}

.link:focus {
  outline: none; /* 기본 포커스 제거 */
  box-shadow: 0 0 0 3px rgba(0, 123, 255, 0.5); /* 포커스 링 추가 */
}

  .spinner {
  position: fixed;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  width: 50px;
  height: 50px;
  border: 5px solid rgba(0, 0, 0, 0.1);
  border-top-color: #007bff;
  border-radius: 50%;
  animation: spin 1s linear infinite;
  z-index: 1000;
  display: none; /* 기본적으로 숨김 */
}

.spinner.active {
  display: block;
}

@keyframes spin {
  0% {
    transform: translate(-50%, -50%) rotate(0deg); /* 중심에서 회전 시작 */
  }
  100% {
    transform: translate(-50%, -50%) rotate(360deg); /* 중심에서 360도 회전 */
  }
}


  </style>
  
<Link to="/photo?id={userId}"><div class="link">내 웹 전시 페이지 들어가기</div></Link>


  <h2>업로드된 이미지 확인 및 삭제</h2>
  <div class="gallery">
    {#each photos as photo}
      <button 
        class="photo" 
        on:click={() => deletePhoto(photo.img_id)}>클릭시 이 이미지 삭제
      
        <img src={photo.src} alt={photo.title} />
        <div class="photo-title">{photo.title}</div>
      </button>
    {/each}
  </div>

  <!-- 로딩 스피너 -->
<div class="spinner {isLoading ? 'active' : ''}"></div>
