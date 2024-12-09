<script>
    const handleGoogleLogin = () => {
      // Google OAuth URL로 리디렉션
      window.location.href = 'https://accounts.google.com/o/oauth2/v2/auth?'
      +'client_id=519458496829-hsgmco71ibrfm1m2vgvh363q6i4t9mos.apps.googleusercontent.com&'
      +`redirect_uri=${window.location.origin}/auth/google/callback&`
      +'response_type=code&'
     +'scope=email';        };

     import { onMount } from 'svelte';

let isAuthenticated = false;

onMount(async () => {
 
  try {
    const token = sessionStorage.getItem('token');
 
 if (!token) {
   isAuthenticated = false; // 토큰이 없으면 인증 실패
   return;
 }

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

$: if (isAuthenticated) {
  window.location.href = '/admin';
}

  </script>
  
  <style>

  
    .container {
      display: flex;
      flex-direction: column;
      min-height: 100vh;
      justify-content: center;
      align-items: center;
      text-align: center;
      padding: 20px;
    }
  
    .card {
      background: #ffffff;
      border-radius: 8px;
      box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
      max-width: 400px;
      width: 100%;
      padding: 30px;
    }
  
    .title {
      font-size: 24px;
      font-weight: bold;
      color: #333333;
      margin-bottom: 20px;
    }
  
    .description {
      color: #666666;
      font-size: 14px;
      margin-bottom: 30px;
    }
  
    .google-btn {
      display: flex;
      align-items: center;
      justify-content: center;
      background: #4285f4;
      color: white;
      font-size: 16px;
      font-weight: bold;
      border: none;
      border-radius: 4px;
      padding: 12px 16px;
      cursor: pointer;
      text-decoration: none;
      transition: background 0.3s ease;
    }
  
    .google-btn:hover {
      background: #357ae8;
    }
  
    .google-btn svg {
      width: 20px;
      height: 20px;
      margin-right: 10px;
    }
  
    footer {
      margin-top: auto;
      padding: 10px;
      background: #333333;
      color: white;
      text-align: center;
      font-size: 14px;
    }
  
    footer a {
      color: #ffcc00;
      text-decoration: none;
    }
  
    footer a:hover {
      text-decoration: underline;
    }
  </style>
  

  <div class="container">
    <!-- 카드 컨테이너 -->
    <div class="card">
      <div class="title">온라인 전시</div>
      <div class="description">로그인하세요</div>
      <button class="google-btn" on:click={handleGoogleLogin}>
        <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 48 48">
          <path fill="#EA4335" d="M24 9.5c3.8 0 7 1.5 9.6 3.8l7.2-7.2C36.6 2 30.7 0 24 0 14.6 0 6.4 5.4 2.4 13.2l8.6 6.7C13.3 13 18.3 9.5 24 9.5z"></path>
          <path fill="#34A853" d="M46.5 24c0-1.7-.2-3.3-.5-4.8H24v9.5h12.7c-.6 3.3-2.6 6.1-5.6 8l8.6 6.7c5-4.6 7.8-11.5 7.8-19.4z"></path>
          <path fill="#4A90E2" d="M13 27.4c-1.2-2.3-1.9-4.9-1.9-7.4s.7-5.1 1.9-7.4L4.4 6.9C0.2 12.7 0 20.3 0 24s.2 11.3 4.4 17.1l8.6-6.7z"></path>
          <path fill="#FBBC05" d="M24 48c6.5 0 12-2.1 16.1-5.7l-8.6-6.7c-2.4 1.6-5.5 2.6-8.9 2.6-5.7 0-10.7-3.6-12.8-8.7l-8.6 6.7C6.4 42.6 14.6 48 24 48z"></path>
        </svg>
구글로 로그인
      </button>
    </div>
  
    <!-- 푸터 -->
    <footer>
      &copy; 2024 My App. <a href="/privacy-policy">Privacy Policy</a>
    </footer>
  </div>
  