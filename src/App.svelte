<script>
	import { Router, Route } from "svelte-routing";
	import Login from './routes/Login.svelte';
	import Photo from './routes/Photo.svelte';
	import Admin from './routes/Admin.svelte';
	import NotFound from './routes/NotFound.svelte';
  </script>
  
  <Router>
	<Route path="/" component={Login} />
	<Route path="/photo" component={Photo} />
	<Route path="/admin" component={Admin} />
	<Route path="*" component={NotFound} />
  </Router>
  