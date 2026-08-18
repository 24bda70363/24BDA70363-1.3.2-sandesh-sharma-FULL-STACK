# 24BDA70363-1.3.2-sandesh-sharma-FULL-STACK
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Role Based Access Control Experiment</title>
<style>
*{box-sizing:border-box}
body{margin:0;font-family:Arial,sans-serif;background:linear-gradient(135deg,#eef2ff,#f8fafc);color:#172033;min-height:100vh;display:flex;align-items:center;justify-content:center;padding:20px}
.card{width:min(100%,620px);background:#fff;border-radius:20px;padding:30px;box-shadow:0 15px 45px #00000018}
h1{margin:10px 0 5px}.sub{color:#667085}.badge{display:inline-block;background:#eef2ff;color:#4338ca;padding:7px 11px;border-radius:20px;font-size:12px;font-weight:bold}
.tabs{display:flex;gap:8px;margin:24px 0}.tab{flex:1;padding:12px;border:0;border-radius:9px;cursor:pointer}.active{background:#4f46e5;color:white}
form{display:grid;gap:9px}label{font-weight:bold;margin-top:6px}input,select{padding:13px;border:1px solid #d0d5dd;border-radius:9px;font-size:15px}
button.primary,button.secondary,.route-btn{width:100%;padding:13px;border:0;border-radius:9px;margin-top:12px;cursor:pointer;font-size:15px}
.primary{background:#4f46e5;color:white}.secondary{background:#eef0f4}.route-btn{background:#111827;color:white}
.message{min-height:24px;margin-top:15px;font-weight:bold}.success{color:#087443}.error{color:#c1121f}
.hidden{display:none}.profile{background:#f8fafc;border:1px solid #e4e7ec;border-radius:12px;padding:18px;margin:20px 0}
.routes{display:grid;grid-template-columns:1fr;gap:10px}.route{border:1px solid #e4e7ec;border-radius:12px;padding:15px}
.route h3{margin:0 0 5px}.route p{color:#667085;margin:5px 0}
.allowed{border-left:5px solid #12b76a}.denied{border-left:5px solid #f04438}
pre{background:#101828;color:#d1fadf;padding:15px;border-radius:10px;white-space:pre-wrap;overflow:auto}
.info{border-top:1px solid #eaecf0;margin-top:24px;padding-top:15px}.info li{margin:8px 0}
</style>
</head>
<body>

<div class="card">
<span class="badge">COLLEGE EXPERIMENT</span>
<h1>Role-Based Access Control</h1>
<p class="sub">Secure application routes based on user permissions</p>

<div id="auth">
  <div class="tabs">
    <button class="tab active" id="loginTab" onclick="setMode('login')">Login</button>
    <button class="tab" id="registerTab" onclick="setMode('register')">Register</button>
  </div>

  <form onsubmit="submitAuth(event)">
    <div id="nameBox" class="hidden">
      <label>Name</label>
      <input id="name" placeholder="Enter your name">
    </div>

    <label>Email</label>
    <input id="email" type="email" placeholder="student@example.com" required>

    <label>Password</label>
    <input id="password" type="password" placeholder="Minimum 6 characters" minlength="6" required>

    <div id="roleBox" class="hidden">
      <label>Select Role</label>
      <select id="role">
        <option value="student">Student</option>
        <option value="teacher">Teacher</option>
        <option value="admin">Admin</option>
      </select>
    </div>

    <button class="primary" id="submitBtn">Login</button>
  </form>
</div>

<div id="dashboard" class="hidden">
  <h2>🔐 RBAC Dashboard</h2>
  <p>Your access is controlled according to your assigned role.</p>

  <div class="profile">
    <h3>Logged-in User</h3>
    <p><b>Name:</b> <span id="userName"></span></p>
    <p><b>Email:</b> <span id="userEmail"></span></p>
    <p><b>Role:</b> <span id="userRole"></span></p>
    <p><b>JWT:</b> <span id="tokenStatus">Active</span></p>
  </div>

  <h3>Application Routes</h3>

  <div class="routes">
    <div class="route allowed">
      <h3>Student Route</h3>
      <p>Permission: student, teacher, admin</p>
      <button class="route-btn" onclick="accessRoute('student')">Open Student Route</button>
    </div>

    <div class="route">
      <h3>Teacher Route</h3>
      <p>Permission: teacher, admin</p>
      <button class="route-btn" onclick="accessRoute('teacher')">Open Teacher Route</button>
    </div>

    <div class="route">
      <h3>Admin Route</h3>
      <p>Permission: admin only</p>
      <button class="route-btn" onclick="accessRoute('admin')">Open Admin Route</button>
    </div>
  </div>

  <pre id="result">Select a route to test role-based authorization.</pre>

  <button class="secondary" onclick="logout()">Logout</button>
</div>

<div id="message" class="message"></div>

<div class="info">
<h3>Experiment Concepts</h3>
<ul>
<li>Authentication using JWT</li>
<li>Role-Based Access Control (RBAC)</li>
<li>Authorization using user roles</li>
<li>Protected application routes</li>
<li>Permission-based resource access</li>
</ul>
</div>
</div>

<script>
let mode="login";
let token=localStorage.getItem("rbac_jwt");
let currentUser=JSON.parse(localStorage.getItem("rbac_user")||"null");

const permissions={
  student:["student"],
  teacher:["student","teacher"],
  admin:["student","teacher","admin"]
};

function setMode(m){
  mode=m;
  document.getElementById("loginTab").classList.toggle("active",m==="login");
  document.getElementById("registerTab").classList.toggle("active",m==="register");
  document.getElementById("nameBox").classList.toggle("hidden",m!=="register");
  document.getElementById("roleBox").classList.toggle("hidden",m!=="register");
  document.getElementById("submitBtn").textContent=m==="register"?"Create Account":"Login";
  showMessage("");
}

function showMessage(text,type=""){
  const el=document.getElementById("message");
  el.textContent=text;
  el.className="message "+type;
}

function encode(obj){
  return btoa(JSON.stringify(obj)).replace(/\+/g,"-").replace(/\//g,"_").replace(/=+$/,"");
}

function createJWT(user){
  const header={alg:"HS256",typ:"JWT"};
  const payload={
    sub:user.id,
    name:user.name,
    email:user.email,
    role:user.role,
    iat:Math.floor(Date.now()/1000),
    exp:Math.floor(Date.now()/1000)+3600
  };
  return encode(header)+"."+encode(payload)+".RBAC_DEMO_SIGNATURE";
}

function decodeJWT(t){
  try{
    const p=t.split(".")[1];
    return JSON.parse(atob(p.replace(/-/g,"+").replace(/_/g,"/")));
  }catch(e){return null}
}

function submitAuth(e){
  e.preventDefault();

  const email=document.getElementById("email").value.trim().toLowerCase();
  const password=document.getElementById("password").value;
  let users=JSON.parse(localStorage.getItem("rbac_users")||"[]");

  if(password.length<6){
    showMessage("Password must contain at least 6 characters.","error");
    return;
  }

  if(mode==="register"){
    const name=document.getElementById("name").value.trim();
    const role=document.getElementById("role").value;

    if(name.length<2){
      showMessage("Please enter a valid name.","error");
      return;
    }

    if(users.some(u=>u.email===email)){
      showMessage("User already exists. Please login.","error");
      return;
    }

    const user={
      id:"USR-"+Date.now(),
      name:name,
      email:email,
      password:password,
      role:role
    };

    users.push(user);
    localStorage.setItem("rbac_users",JSON.stringify(users));
    authenticate(user);
    showMessage("Registration successful. JWT created.","success");
  }else{
    const user=users.find(u=>u.email===email && u.password===password);

    if(!user){
      showMessage("Invalid email or password.","error");
      return;
    }

    authenticate(user);
    showMessage("Login successful. Role permissions loaded.","success");
  }
}

function authenticate(user){
  currentUser=user;
  token=createJWT(user);

  localStorage.setItem("rbac_user",JSON.stringify(user));
  localStorage.setItem("rbac_jwt",token);

  document.getElementById("auth").classList.add("hidden");
  document.getElementById("dashboard").classList.remove("hidden");

  document.getElementById("userName").textContent=user.name;
  document.getElementById("userEmail").textContent=user.email;
  document.getElementById("userRole").textContent=user.role.toUpperCase();
  document.getElementById("tokenStatus").textContent="Active";
}

function accessRoute(requiredRole){
  const payload=decodeJWT(token);

  if(!payload){
    showMessage("Invalid authentication token.","error");
    logout();
    return;
  }

  const now=Math.floor(Date.now()/1000);

  if(payload.exp<now){
    showMessage("JWT expired. Login again.","error");
    logout();
    return;
  }

  const allowed=permissions[payload.role] || [];

  if(!allowed.includes(requiredRole)){
    document.getElementById("result").textContent=
      "ACCESS DENIED\n\n"+
      "Requested Route: "+requiredRole.toUpperCase()+"\n"+
      "Your Role: "+payload.role.toUpperCase()+"\n"+
      "Required Permission: "+requiredRole+"\n\n"+
      "You do not have permission to access this route.";

    showMessage("Access denied: insufficient permissions.","error");
    return;
  }

  document.getElementById("result").textContent=
    "ACCESS GRANTED\n\n"+
    "Requested Route: "+requiredRole.toUpperCase()+"\n"+
    "Authenticated User: "+payload.name+"\n"+
    "User Role: "+payload.role.toUpperCase()+"\n"+
    "Permission: "+requiredRole+"\n\n"+
    "Role-based authorization successful.";

  showMessage("Access granted to protected "+requiredRole+" route.","success");
}

function logout(){
  token=null;
  currentUser=null;
  localStorage.removeItem("rbac_jwt");
  localStorage.removeItem("rbac_user");
  document.getElementById("dashboard").classList.add("hidden");
  document.getElementById("auth").classList.remove("hidden");
  document.getElementById("result").textContent="Select a route to test role-based authorization.";
  setMode("login");
  showMessage("Logged out successfully.","success");
}

if(token && currentUser){
  authenticate(currentUser);
}
</script>
</body>
</html>
