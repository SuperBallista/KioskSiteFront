<script>
   import { onMount } from 'svelte';
   import Upload from './components/Upload.svelte';
   
   let isAuthenticated = null; // 초기 로딩 상태
 
   onMount(async () => {
     // URL에서 토큰 추출
     const urlParams = new URLSearchParams(window.location.search);
     const tokenFromUrl = urlParams.get('token');
 
     if (tokenFromUrl) {
       // URL에서 가져온 토큰을 세션스토리지에 저장
       sessionStorage.setItem('token', tokenFromUrl);
 
       // URL 정리 (토큰 제거)
       const url = new URL(window.location.href);
       url.searchParams.delete('token');
       window.history.replaceState({}, document.title, url.toString());
     }
 
     // 세션스토리지에서 토큰 가져오기
     const token = sessionStorage.getItem('token');
 
     if (!token) {
       isAuthenticated = false; // 토큰이 없으면 인증 실패
       return;
     }
 
     try {
       const response = await fetch('/auth/validate', {
         method: 'POST', // 토큰 유효성 확인 요청 (POST 추천)
         headers: {
           'Content-Type': 'application/json',
         },
         body: JSON.stringify({ token }), // 토큰 전달
       });
 
       if (response.ok) {
         const result = await response.json();
         isAuthenticated = result.valid; // 결과에 따라 인증 상태 설정
       } else {
         isAuthenticated = false; // 인증 실패
       }
     } catch (error) {
       console.error('Token validation failed:', error);
       isAuthenticated = false; // 에러 발생 시 인증 실패로 간주
     }
   });
 
   // 인증 실패 시 리다이렉트
   $: {
     if (isAuthenticated === false) {
       window.location.href = '/';
     }
   }
 </script>
 
 {#if isAuthenticated === true}
   <header>
    <h1>웹 전시 페이지를 관리합니다</h1>
   </header>
<main>
  <Upload />
</main>
<footer>
  
</footer>

 {:else if isAuthenticated === null}
   <p>Loading...</p>
 {/if}
 