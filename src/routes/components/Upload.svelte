<div id="dropzone" class="dropzone">
  <!-- 기본 드래그 앤 드롭 메시지 -->
  <div class="dz-message">
    <button class="upload-button">업로드할 파일 선택하기</button>
    <p>여기에 파일을 드래그하거나 클릭하세요.</p>
  </div>
</div>

<style>
  /* Dropzone 영역 스타일 */
  .dropzone {
    border: 2px dashed #007bff;
    border-radius: 8px;
    padding: 20px;
    min-height: 200px;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    background-color: #f8f9fa;
    transition: background-color 0.3s ease, border-color 0.3s ease;
  }

  /* 드래그 시 스타일 변경 */
  .dropzone.dz-drag-hover {
    background-color: #e9f5ff;
    border-color: #0056b3;
  }

  /* 기본 메시지 */
  .dz-message {
    text-align: center;
  }

  .dz-message p {
    margin: 10px 0;
    font-size: 14px;
    color: #6c757d;
  }

  /* 업로드 버튼 스타일 */
  .upload-button, .logout-button {
    background-color: #007bff;
    color: white;
    border: none;
    padding: 10px 20px;
    border-radius: 20px;
    font-size: 16px;
    cursor: pointer;
    transition: background-color 0.3s ease, box-shadow 0.3s ease;
  }

  .upload-button:hover, .logout-button:hover {
    background-color: #0056b3;
    box-shadow: 0 4px 10px rgba(0, 0, 0, 0.1);
  }

  /* 삭제 버튼 스타일 */
  .custom-remove-button {
    position: absolute;
    top: 5px;
    right: 5px;
    background: #dc3545;
    color: white;
    border: none;
    padding: 5px 8px;
    border-radius: 50%;
    font-size: 14px;
    cursor: pointer;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
    transition: background-color 0.3s ease, transform 0.2s ease;
  }

  .custom-remove-button:hover {
    background-color: #bd2130;
    transform: scale(1.1);
  }

  /* 미리보기 아이템 */
  .custom-preview-container {
    display: flex;
    flex-wrap: wrap;
    gap: 10px;
    margin-top: 20px;
  }

  .custom-preview-item {
    flex: 1 0 calc(25% - 10px);
    max-width: calc(25% - 10px);
    position: relative;
    border: 1px solid #ddd;
    border-radius: 8px;
    overflow: hidden;
    display: flex;
    align-items: center;
    justify-content: center;
    background-color: #fff;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
    transition: transform 0.3s ease, box-shadow 0.3s ease;
  }

  .custom-preview-item:hover {
    transform: scale(1.05);
    box-shadow: 0 4px 10px rgba(0, 0, 0, 0.2);
  }

  .custom-preview-item img {
    width: 100%;
    height: auto;
    display: block;
  }  /* 로딩 스피너 스타일 */
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
<script>
  import { onMount } from "svelte";
  import Dropzone from "dropzone";
  import ImgAdmin from "./ImgAdmin.svelte";

  let dropzone;
  let refreshKey = 0; // 새로고침을 트리거할 키
  let isLoading = false; // 로딩 상태

  // 업로드 대기 중인 파일이 있는지 확인하는 함수
  function processQueue() {
    if (dropzone.getQueuedFiles().length > 0) {
      dropzone.processQueue(); // 파일 업로드 실행
    } else {
      alert("업로드할 파일을 선택하세요!");
    }
  }

  function logout() {
    sessionStorage.removeItem("token");
    window.location.href = "/";
  }


  onMount(() => {
    dropzone = new Dropzone("#dropzone", {
      maxFiles: 20, // 업로드 가능한 파일 수를 적절히 증가
      parallelUploads: 20, // 동시에 전송 가능한 파일 수 증가
      url: "/photo/upload", // 서버 업로드 엔드포인트
      autoProcessQueue: false, // 자동 업로드 비활성화
      headers: {
        Authorization: `Bearer ${sessionStorage.getItem("token")}`, // 토큰 추가
      },
      previewsContainer: ".custom-preview-container", // 커스텀 미리보기 컨테이너
      dictDefaultMessage: "",
      previewTemplate: `
        <div class="custom-preview-item">
          <img data-dz-thumbnail />
          <button class="custom-remove-button" data-dz-remove>✖</button>
        </div>
      `,
      init: function () {
        this.on("sending", () => {
          isLoading = true; // 로딩 상태 활성화
        });

        this.on("queuecomplete", () => {
          isLoading = false; // 로딩 상태 비활성화
          dropzone.removeAllFiles();
          refreshKey++;
        });

        this.on("sending", (file, xhr, formData) => {
          // 파일 이름에서 확장자를 제거
          const fileNameWithoutExtension = file.name.replace(/\.[^/.]+$/, ""); // 확장자 제거

          // 파일 이름을 UTF-8로 인코딩
          const encodedFileName = encodeURIComponent(fileNameWithoutExtension);

          // 헤더에 인코딩된 파일 이름 설정
          xhr.setRequestHeader("FileTitle", encodedFileName);
        });
      },
    });

    // 업로드 완료 시 Dropzone 정리 및 ImgAdmin 새로고침
    dropzone.on("queuecomplete", () => {
      dropzone.removeAllFiles(); // 모든 파일 제거
      refreshKey++; // 키 값 변경으로 ImgAdmin 새로고침
    });

    return () => dropzone.destroy(); // 컴포넌트 언마운트 시 Dropzone 정리
  });
</script>

<div class="custom-preview-container"></div>

<!-- 업로드 버튼 -->
<div class="upload-controls">
  <button on:click={processQueue} class="upload-button">선택한 파일 업로드하기</button>
</div>


<!-- 로딩 스피너 -->
<div class="spinner {isLoading ? 'active' : ''}"></div>

<!-- ImgAdmin 컴포넌트 -->
{#key refreshKey}
  <ImgAdmin />
{/key}

<button class="logout-button" on:click={() => logout()}>로그아웃</button>
