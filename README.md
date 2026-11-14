# Pos
<application>
  <html lang="en">
  <head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Multi-Tenant POS SaaS Platform</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <script>
      tailwind.config = {
        theme: {
          extend: {
            colors: {
              brand: {
                50: '#eef2ff',
                100: '#e0e7ff',
                500: '#6366f1',
                600: '#4f46e5',
                700: '#4338ca',
                900: '#312e81'
              }
            }
          }
        }
      }
    </script>
  </head>
  <body class="bg-slate-100 text-slate-800 font-sans antialiased h-screen flex flex-col overflow-hidden">

    <!-- ROOT CONTAINER -->
    <div id="app" class="flex-1 flex flex-col h-full overflow-hidden">
      <!-- RENDERED DYNAMICALLY VIA JS -->
    </div>

    <!-- SCRIPT LOGIC -->
    <script>
      // --- STATE MANAGEMENT ---
      const state = {
        currentView: 'login', // 'login', 'admin_dashboard', 'company_dashboard', 'counter_dashboard'
        currentUser: null,
        
        // Admin Data
        adminCredentials: {
          email: 'choyjiez07@gmail.com',
          pass: 'ChoyJiez#0716'
        },
        subscriptions: [
          { id: 1, name: 'Standard 14-Day Free Trial', price: 0, duration: '14 Days', status: 'Active' },
          { id: 2, name: 'Monthly Pro Plan', price: 1500, duration: '30 Days', status: 'Active' },
          { id: 3, name: 'Enterprise Annual Tier', price: 15000, duration: '365 Days', status: 'Active' }
        ],
        companies: [
          { id: 1, name: 'Brew & Bake Café', owner: 'Maria Santos', email: 'maria@brew.com', status: 'Approved', trialDaysLeft: 10, plan: 'Monthly Pro Plan' },
          { id: 2, name: 'Urban Retail Hub', owner: 'Juan Dela Cruz', email: 'juan@urban.com', status: 'Pending Approval', trialDaysLeft: 14, plan: 'Standard 14-Day Free Trial' },
          { id: 3, name: 'FastGrill Bistro', owner: 'Ana Reyes', email: 'ana@fastgrill.com', status: 'Suspended', trialDaysLeft: 0, plan: 'Monthly Pro Plan' }
        ],

        // Company Manager Data
        companyManagerAuth: {
          email: 'manager@pos.com',
          pass: 'manager123'
        },
        products: [
          { id: 1, name: 'Espresso Roast', category: 'Coffee', price: 120, stock: 45, unit: 'PCS' },
          { id: 2, name: 'Caramel Macchiato', category: 'Coffee', price: 180, stock: 30, unit: 'PCS' },
          { id: 3, name: 'Butter Croissant', category: 'Pastries', price: 95, stock: 15, unit: 'PCS' },
          { id: 4, name: 'Cheesecake Slice', category: 'Pastries', price: 150, stock: 8, unit: 'PCS' },
          { id: 5, name: 'Club Sandwich', category: 'Meals', price: 210, stock: 20, unit: 'PCS' }
        ],
        managerCode: '1234', // required for voids, discounts, cash-in/out
        salesHistory: [
          { id: 'INV-1001', cashier: 'Counter 1', total: 300, items: 2, time: '10:45 AM', status: 'Completed' },
          { id: 'INV-1002', cashier: 'Counter 1', total: 210, items: 1, time: '11:15 AM', status: 'Completed' }
        ],

        // Counter Dashboard Cart & Drawer State
        cart: [],
        cashDrawer: {
          isOpen: true,
          balance: 5000,
          shiftStarted: '08:00 AM'
        },
        activeCategory: 'All',
        
        // Modals & Notifications
        modal: { type: null, data: null },
        notification: null
      };

      // --- HELPER NOTIFICATION ---
      function showNotification(msg, type = 'success') {
        state.notification = { msg, type };
        render();
        setTimeout(() => {
          state.notification = null;
          render();
        }, 3000);
      }

      // --- ROUTER & RENDER ENGINE ---
      function render() {
        const app = document.getElementById('app');
        app.innerHTML = '';

        // Render Notification Toast if active
        let notificationHTML = '';
        if (state.notification) {
          const bg = state.notification.type === 'error' ? 'bg-red-600' : 'bg-emerald-600';
          notificationHTML = `
            <div class="fixed top-5 right-5 z-50 px-4 py-3 rounded-xl text-white shadow-xl ${bg} flex items-center space-x-3 transition transform duration-300">
              <i class="fa-solid fa-circle-info text-lg"></i>
              <span class="font-medium">${state.notification.msg}</span>
            </div>
          `;
        }

        if (state.currentView === 'login') {
          app.innerHTML = renderLoginView() + notificationHTML;
        } else if (state.currentView === 'admin_dashboard') {
          app.innerHTML = renderAdminDashboard() + notificationHTML;
        } else if (state.currentView === 'company_dashboard') {
          app.innerHTML = renderCompanyDashboard() + notificationHTML;
        } else if (state.currentView === 'counter_dashboard') {
          app.innerHTML = renderCounterDashboard() + notificationHTML;
        }

        attachEventListeners();
      }

      // --- 1. LOGIN VIEW ---
      function renderLoginView() {
        return `
          <div class="flex-1 flex items-center justify-center bg-gradient-to-br from-indigo-900 via-slate-900 to-indigo-950 p-6">
            <div class="bg-white/95 backdrop-blur-md rounded-2xl shadow-2xl w-full max-w-md p-8 border border-white/20">
              <div class="text-center mb-8">
                <div class="inline-flex items-center justify-center w-16 h-16 bg-brand-600 text-white rounded-2xl shadow-lg mb-4 text-2xl font-bold">
                  <i class="fa-solid fa-cash-register"></i>
                </div>
                <h1 class="text-2xl font-bold text-slate-900">Multi-Tenant POS SaaS</h1>
                <p class="text-sm text-slate-500 mt-1">Sign in to your Super Admin or Company Account</p>
              </div>

              <form id="login-form" class="space-y-5">
                <div>
                  <label class="block text-xs font-semibold uppercase tracking-wider text-slate-600 mb-1">Email Address</label>
                  <div class="relative">
                    <span class="absolute inset-y-0 left-0 pl-3 flex items-center text-slate-400"><i class="fa-solid fa-envelope"></i></span>
                    <input type="email" id="login-email" required value="choyjiez07@gmail.com" class="w-full pl-10 pr-4 py-2.5 bg-slate-50 border border-slate-300 rounded-xl text-sm focus:ring-2 focus:ring-brand-500 focus:outline-none">
                  </div>
                </div>

                <div>
                  <label class="block text-xs font-semibold uppercase tracking-wider text-slate-600 mb-1">Password</label>
                  <div class="relative">
                    <span class="absolute inset-y-0 left-0 pl-3 flex items-center text-slate-400"><i class="fa-solid fa-lock"></i></span>
                    <input type="password" id="login-pass" required value="ChoyJiez#0716" class="w-full pl-10 pr-4 py-2.5 bg-slate-50 border border-slate-300 rounded-xl text-sm focus:ring-2 focus:ring-brand-500 focus:outline-none">
                  </div>
                </div>

                <div class="pt-2">
                  <button type="submit" class="w-full bg-brand-600 hover:bg-brand-700 text-white py-3 rounded-xl font-semibold shadow-lg shadow-brand-500/30 transition">
                    Sign In
                  </button>
                </div>
              </form>

              <div class="mt-6 border-t border-slate-200 pt-5 text-center text-xs text-slate-500">
                <p class="font-medium mb-2">Quick Demo Access:</p>
                <div class="flex justify-center gap-2">
                  <button type="button" onclick="fillAdminCreds()" class="px-3 py-1.5 bg-indigo-50 text-indigo-700 rounded-lg font-medium hover:bg-indigo-100 transition">Admin Demo</button>
                  <button type="button" onclick="fillCompanyCreds()" class="px-3 py-1.5 bg-emerald-50 text-emerald-700 rounded-lg font-medium hover:bg-emerald-100 transition">Company Demo</button>
                </div>
              </div>
            </div>
          </div>
        `;
      }

      // --- 2. ADMIN DASHBOARD ---
      let adminTab = 'subscriptions'; // 'subscriptions', 'companies', 'approvals'

      function renderAdminDashboard() {
        return `
          <div class="flex-1 flex h-full overflow-hidden bg-slate-100">
            <!-- Sidebar -->
            <aside class="w-64 bg-slate-900 text-slate-300 flex flex-col justify-between border-r border-slate-800">
              <div>
                <div class="p-6 border-b border-slate-800 flex items-center space-x-3">
                  <div class="w-10 h-10 bg-brand-600 rounded-xl flex items-center justify-center text-white font-bold">
                    <i class="fa-solid fa-shield-halved"></i>
                  </div>
                  <div>
                    <h2 class="font-bold text-white text-sm">Super Admin</h2>
                    <span class="text-xs text-emerald-400 flex items-center gap-1"><i class="fa-solid fa-circle text-[8px]"></i> Online</span>
                  </div>
                </div>
                <nav class="p-4 space-y-1.5">
                  <button onclick="setAdminTab('subscriptions')" class="w-full flex items-center space-x-3 px-4 py-3 rounded-xl text-sm font-medium transition ${adminTab === 'subscriptions' ? 'bg-brand-600 text-white shadow-lg' : 'hover:bg-slate-800 text-slate-400'}">
                    <i class="fa-solid fa-tags w-5"></i>
                    <span>Pricing & Subscriptions</span>
                  </button>
                  <button onclick="setAdminTab('companies')" class="w-full flex items-center space-x-3 px-4 py-3 rounded-xl text-sm font-medium transition ${adminTab === 'companies' ? 'bg-brand-600 text-white shadow-lg' : 'hover:bg-slate-800 text-slate-400'}">
                    <i class="fa-solid fa-building w-5"></i>
                    <span>Company List</span>
                  </button>
                  <button onclick="setAdminTab('approvals')" class="w-full flex items-center space-x-3 px-4 py-3 rounded-xl text-sm font-medium transition ${adminTab === 'approvals' ? 'bg-brand-600 text-white shadow-lg' : 'hover:bg-slate-800 text-slate-400'}">
                    <i class="fa-solid fa-user-check w-5"></i>
                    <span>Company Approvals</span>
                  </button>
                </nav>
              </div>
              <div class="p-4 border-t border-slate-800">
                <button onclick="logout()" class="w-full flex items-center space-x-3 px-4 py-3 rounded-xl text-sm font-medium text-red-400 hover:bg-red-500/10 transition">
                  <i class="fa-solid fa-arrow-right-from-bracket w-5"></i>
                  <span>Sign Out</span>
                </button>
              </div>
            </aside>

            <!-- Main Content Area -->
            <main class="flex-1 flex flex-col overflow-hidden">
              <header class="h-16 bg-white border-b border-slate-200 px-8 flex items-center justify-between shadow-sm">
                <h1 class="text-lg font-bold text-slate-800 capitalize"><i class="fa-solid fa-gauge-high mr-2 text-brand-600"></i> Admin Control Center - ${adminTab}</h1>
                <div class="flex items-center space-x-3">
                  <button onclick="switchView('company_dashboard')" class="px-3 py-1.5 bg-slate-100 hover:bg-slate-200 text-slate-700 text-xs font-semibold rounded-lg transition">Switch to Company View</button>
                  <span class="text-xs font-semibold bg-indigo-50 text-indigo-700 px-3 py-1 rounded-full">choyjiez07@gmail.com</span>
                </div>
              </header>

              <div class="flex-1 overflow-y-auto p-8">
                ${adminTab === 'subscriptions' ? renderAdminSubscriptions() : ''}
                ${adminTab === 'companies' ? renderAdminCompanies() : ''}
                ${adminTab === 'approvals' ? renderAdminApprovals() : ''}
              </div>
            </main>
          </div>
        `;
      }

      function renderAdminSubscriptions() {
        return `
          <div class="max-w-5xl mx-auto space-y-6">
            <div class="flex justify-between items-center">
              <div>
                <h2 class="text-xl font-bold text-slate-900">Subscription & Pricing Management</h2>
                <p class="text-sm text-slate-500">Configure tenant subscription plans, trial periods, and tier pricing amounts.</p>
              </div>
            </div>

            <div class="grid grid-cols-1 md:grid-cols-3 gap-6">
              ${state.subscriptions.map(sub => `
                <div class="bg-white rounded-2xl shadow-sm border border-slate-200 p-6 flex flex-col justify-between">
                  <div>
                    <div class="flex justify-between items-start mb-4">
                      <span class="text-xs font-bold uppercase tracking-wider px-2.5 py-1 bg-indigo-50 text-indigo-700 rounded-lg">${sub.duration}</span>
                      <span class="text-xs font-semibold text-emerald-600 bg-emerald-50 px-2.5 py-1 rounded-full">${sub.status}</span>
                    </div>
                    <h3 class="text-lg font-bold text-slate-900 mb-1">${sub.name}</h3>
                    <div class="text-2xl font-extrabold text-slate-900 my-4">₱${sub.price.toLocaleString()} <span class="text-xs font-normal text-slate-500">/ billing cycle</span></div>
                  </div>
                  <button onclick="openEditSubscriptionModal(${sub.id})" class="mt-6 w-full py-2.5 bg-slate-900 hover:bg-brand-600 text-white font-semibold rounded-xl text-sm transition shadow-md">
                    <i class="fa-solid fa-pen-to-square mr-2"></i> Edit Subscription Pricing
                  </button>
                </div>
              `).join('')}
            </div>
          </div>
        `;
      }

      function renderAdminCompanies() {
        return `
          <div class="max-w-6xl mx-auto space-y-6">
            <div class="flex justify-between items-center">
              <div>
                <h2 class="text-xl font-bold text-slate-900">Registered Companies Directory</h2>
                <p class="text-sm text-slate-500">Monitor all tenant companies, active subscriptions, and trial statuses.</p>
              </div>
            </div>

            <div class="bg-white rounded-2xl shadow-sm border border-slate-200 overflow-hidden">
              <table class="w-full text-left border-collapse">
                <thead>
                  <tr class="bg-slate-50 border-b border-slate-200 text-xs font-semibold text-slate-600 uppercase tracking-wider">
                    <th class="p-4">Company Name</th>
                    <th class="p-4">Owner / Email</th>
                    <th class="p-4">Plan & Trial</th>
                    <th class="p-4">Status</th>
                    <th class="p-4 text-right">Actions</th>
                  </tr>
                </thead>
                <tbody class="divide-y divide-slate-100 text-sm">
                  ${state.companies.map(c => `
                    <tr class="hover:bg-slate-50/50">
                      <td class="p-4 font-bold text-slate-900">${c.name}</td>
                      <td class="p-4">
                        <div class="font-medium text-slate-800">${c.owner}</div>
                        <div class="text-xs text-slate-500">${c.email}</div>
                      </td>
                      <td class="p-4">
                        <div class="font-medium text-slate-800">${c.plan}</div>
                        <div class="text-xs text-indigo-600">${c.trialDaysLeft} Days Trial Remaining</div>
                      </td>
                      <td class="p-4">
                        <span class="px-3 py-1 rounded-full text-xs font-semibold ${c.status === 'Approved' ? 'bg-emerald-50 text-emerald-700' : c.status === 'Pending Approval' ? 'bg-amber-50 text-amber-700' : 'bg-red-50 text-red-700'}">
                          ${c.status}
                        </span>
                      </td>
                      <td class="p-4 text-right">
                        <button onclick="toggleCompanyStatus(${c.id})" class="px-3 py-1.5 bg-slate-100 hover:bg-slate-200 text-slate-700 rounded-lg text-xs font-semibold transition">
                          ${c.status === 'Suspended' ? 'Activate' : 'Suspend'}
                        </button>
                      </td>
                    </tr>
                  `).join('')}
                </tbody>
              </table>
            </div>
          </div>
        `;
      }

      function renderAdminApprovals() {
        const pending = state.companies.filter(c => c.status === 'Pending Approval');
        return `
          <div class="max-w-5xl mx-auto space-y-6">
            <div>
              <h2 class="text-xl font-bold text-slate-900">Company Approval Requests</h2>
              <p class="text-sm text-slate-500">Review pending company registrations and grant access to the POS SaaS ecosystem.</p>
            </div>

            ${pending.length === 0 ? `
              <div class="bg-white rounded-2xl p-12 text-center border border-slate-200 shadow-sm">
                <i class="fa-solid fa-circle-check text-4xl text-emerald-500 mb-3"></i>
                <h3 class="text-lg font-bold text-slate-800">No Pending Approvals</h3>
                <p class="text-sm text-slate-500 mt-1">All company registration requests have been processed successfully.</p>
              </div>
            ` : `
              <div class="grid grid-cols-1 gap-4">
                ${pending.map(c => `
                  <div class="bg-white rounded-2xl p-6 border border-slate-200 shadow-sm flex items-center justify-between">
                    <div>
                      <span class="text-xs font-bold uppercase tracking-wider px-2.5 py-1 bg-amber-50 text-amber-700 rounded-lg">Pending Review</span>
                      <h3 class="text-lg font-bold text-slate-900 mt-2">${c.name}</h3>
                      <p class="text-sm text-slate-600">Owner: <span class="font-medium">${c.owner}</span> (${c.email})</p>
                      <p class="text-xs text-slate-500 mt-1">Requested Plan: ${c.plan} (14-Day Free Trial initiated)</p>
                    </div>
                    <div class="flex space-x-3">
                      <button onclick="approveCompany(${c.id})" class="px-4 py-2 bg-emerald-600 hover:bg-emerald-700 text-white rounded-xl text-sm font-semibold shadow-md transition">
                        <i class="fa-solid fa-check mr-1.5"></i> Approve Company
                      </button>
                    </div>
                  </div>
                `).join('')}
              </div>
            `}
          </div>
        `;
      }


      // --- 3. COMPANY / MANAGER DASHBOARD ---
      let managerTab = 'overview'; // 'overview', 'products', 'employees', 'void_setup'

      function renderCompanyDashboard() {
        return `
          <div class="flex-1 flex h-full overflow-hidden bg-slate-100">
            <!-- Sidebar -->
            <aside class="w-64 bg-slate-900 text-slate-300 flex flex-col justify-between border-r border-slate-800">
              <div>
                <div class="p-6 border-b border-slate-800 flex items-center space-x-3">
                  <div class="w-10 h-10 bg-indigo-600 rounded-xl flex items-center justify-center text-white font-bold">
                    <i class="fa-solid fa-store"></i>
                  </div>
                  <div>
                    <h2 class="font-bold text-white text-sm">Brew & Bake Café</h2>
                    <span class="text-xs text-emerald-400 flex items-center gap-1"><i class="fa-solid fa-clock text-[8px]"></i> 14 Days Free Trial Active</span>
                  </div>
                </div>
                <nav class="p-4 space-y-1.5">
                  <button onclick="setManagerTab('overview')" class="w-full flex items-center space-x-3 px-4 py-3 rounded-xl text-sm font-medium transition ${managerTab === 'overview' ? 'bg-indigo-600 text-white shadow-lg' : 'hover:bg-slate-800 text-slate-400'}">
                    <i class="fa-solid fa-chart-pie w-5"></i>
                    <span>Manager Dashboard</span>
                  </button>
                  <button onclick="setManagerTab('products')" class="w-full flex items-center space-x-3 px-4 py-3 rounded-xl text-sm font-medium transition ${managerTab === 'products' ? 'bg-indigo-600 text-white shadow-lg' : 'hover:bg-slate-800 text-slate-400'}">
                    <i class="fa-solid fa-boxes-stacked w-5"></i>
                    <span>Products & Inventory</span>
                  </button>
                  <button onclick="setManagerTab('employees')" class="w-full flex items-center space-x-3 px-4 py-3 rounded-xl text-sm font-medium transition ${managerTab === 'employees' ? 'bg-indigo-600 text-white shadow-lg' : 'hover:bg-slate-800 text-slate-400'}">
                    <i class="fa-solid fa-users w-5"></i>
                    <span>Employee Accounts</span>
                  </button>
                  <button onclick="setManagerTab('void_setup')" class="w-full flex items-center space-x-3 px-4 py-3 rounded-xl text-sm font-medium transition ${managerTab === 'void_setup' ? 'bg-indigo-600 text-white shadow-lg' : 'hover:bg-slate-800 text-slate-400'}">
                    <i class="fa-solid fa-lock-keyhole w-5"></i>
                    <span>Void & Discount Setup</span>
                  </button>
                </nav>
              </div>
              <div class="p-4 border-t border-slate-800 space-y-2">
                <button onclick="switchView('counter_dashboard')" class="w-full flex items-center space-x-3 px-4 py-2.5 rounded-xl text-sm font-semibold bg-emerald-600 hover:bg-emerald-700 text-white transition shadow-md">
                  <i class="fa-solid fa-cash-register w-5"></i>
                  <span>Launch POS Counter</span>
                </button>
                <button onclick="logout()" class="w-full flex items-center space-x-3 px-4 py-2.5 rounded-xl text-sm font-medium text-red-400 hover:bg-red-500/10 transition">
                  <i class="fa-solid fa-arrow-right-from-bracket w-5"></i>
                  <span>Sign Out</span>
                </button>
              </div>
            </aside>

            <!-- Main Content -->
            <main class="flex-1 flex flex-col overflow-hidden">
              <header class="h-16 bg-white border-b border-slate-200 px-8 flex items-center justify-between shadow-sm">
                <h1 class="text-lg font-bold text-slate-800 capitalize"><i class="fa-solid fa-store mr-2 text-indigo-600"></i> Company Portal - ${managerTab.replace('_', ' ')}</h1>
                <div class="flex items-center space-x-3">
                  <span class="text-xs font-semibold bg-emerald-50 text-emerald-700 px-3 py-1 rounded-full"><i class="fa-solid fa-shield-check mr-1"></i> 14-Day Free Trial (Active)</span>
                  <button onclick="switchView('admin_dashboard')" class="px-3 py-1.5 bg-slate-100 hover:bg-slate-200 text-slate-700 text-xs font-semibold rounded-lg transition">Back to Super Admin</button>
                </div>
              </header>

              <div class="flex-1 overflow-y-auto p-8">
                ${managerTab === 'overview' ? renderManagerOverview() : ''}
                ${managerTab === 'products' ? renderManagerProducts() : ''}
                ${managerTab === 'employees' ? renderManagerEmployees() : ''}
                ${managerTab === 'void_setup' ? renderManagerVoidSetup() : ''}
              </div>
            </main>
          </div>
        `;
      }

      function renderManagerOverview() {
        return `
          <div class="max-w-6xl mx-auto space-y-6">
            <div class="grid grid-cols-1 md:grid-cols-4 gap-6">
              <div class="bg-white p-6 rounded-2xl shadow-sm border border-slate-200">
                <div class="text-xs font-bold text-slate-400 uppercase tracking-wider mb-1">Total Sales Today</div>
                <div class="text-2xl font-extrabold text-slate-900">₱5,240.00</div>
                <div class="text-xs text-emerald-600 mt-2 font-medium"><i class="fa-solid fa-arrow-trend-up"></i> +12% from yesterday</div>
              </div>
              <div class="bg-white p-6 rounded-2xl shadow-sm border border-slate-200">
                <div class="text-xs font-bold text-slate-400 uppercase tracking-wider mb-1">Active Transactions</div>
                <div class="text-2xl font-extrabold text-slate-900">28 Orders</div>
                <div class="text-xs text-indigo-600 mt-2 font-medium">Counter 1 active</div>
              </div>
              <div class="bg-white p-6 rounded-2xl shadow-sm border border-slate-200">
                <div class="text-xs font-bold text-slate-400 uppercase tracking-wider mb-1">Trial Period Status</div>
                <div class="text-2xl font-extrabold text-emerald-600">10 Days Left</div>
                <div class="text-xs text-slate-500 mt-2">Requires subscription post-trial</div>
              </div>
              <div class="bg-white p-6 rounded-2xl shadow-sm border border-slate-200">
                <div class="text-xs font-bold text-slate-400 uppercase tracking-wider mb-1">Active Cash Drawer</div>
                <div class="text-2xl font-extrabold text-slate-900">₱5,000.00</div>
                <div class="text-xs text-emerald-600 mt-2 font-medium">Drawer Open / Verified</div>
              </div>
            </div>

            <!-- Transaction Logs & Counter Logs -->
            <div class="bg-white rounded-2xl shadow-sm border border-slate-200 p-6">
              <h3 class="text-lg font-bold text-slate-900 mb-4">Counter & Transaction Audit Logs</h3>
              <div class="overflow-x-auto">
                <table class="w-full text-left border-collapse text-sm">
                  <thead>
                    <tr class="bg-slate-50 border-b border-slate-200 text-xs font-semibold text-slate-500 uppercase tracking-wider">
                      <th class="p-3">Invoice ID</th>
                      <th class="p-3">Cashier Station</th>
                      <th class="p-3">Items Count</th>
                      <th class="p-3">Total Amount</th>
                      <th class="p-3">Timestamp</th>
                      <th class="p-3">Status</th>
                    </tr>
                  </thead>
                  <tbody class="divide-y divide-slate-100">
                    ${state.salesHistory.map(s => `
                      <tr class="hover:bg-slate-50/50">
                        <td class="p-3 font-bold text-slate-900">${s.id}</td>
                        <td class="p-3 text-slate-700">${s.cashier}</td>
                        <td class="p-3 text-slate-700">${s.items} item(s)</td>
                        <td class="p-3 font-bold text-indigo-600">₱${s.total.toFixed(2)}</td>
                        <td class="p-3 text-slate-500">${s.time}</td>
                        <td class="p-3"><span class="px-2.5 py-1 bg-emerald-50 text-emerald-700 rounded-full text-xs font-semibold">${s.status}</span></td>
                      </tr>
                    `).join('')}
                  </tbody>
                </table>
              </div>
            </div>
          </div>
        `;
      }

      function renderManagerProducts() {
        return `
          <div class="max-w-6xl mx-auto space-y-6">
            <div class="flex justify-between items-center">
              <div>
                <h2 class="text-xl font-bold text-slate-900">Product & Inventory Catalog</h2>
                <p class="text-sm text-slate-500">Products auto-deduct from inventory upon successful POS counter checkout.</p>
              </div>
              <button onclick="openAddProductModal()" class="px-4 py-2 bg-indigo-600 hover:bg-indigo-700 text-white rounded-xl text-sm font-semibold shadow-md transition">
                <i class="fa-solid fa-plus mr-1.5"></i> Add New Product
              </button>
            </div>

            <div class="bg-white rounded-2xl shadow-sm border border-slate-200 overflow-hidden">
              <table class="w-full text-left border-collapse text-sm">
                <thead>
                  <tr class="bg-slate-50 border-b border-slate-200 text-xs font-semibold text-slate-500 uppercase tracking-wider">
                    <th class="p-4">Product Name</th>
                    <th class="p-4">Category</th>
                    <th class="p-4">Price</th>
                    <th class="p-4">Stock Level (Auto-Deduct)</th>
                    <th class="p-4 text-right">Actions</th>
                  </tr>
                </thead>
                <tbody class="divide-y divide-slate-100">
                  ${state.products.map(p => `
                    <tr class="hover:bg-slate-50/50">
                      <td class="p-4 font-bold text-slate-900">${p.name}</td>
                      <td class="p-4"><span class="px-2.5 py-1 bg-indigo-50 text-indigo-700 rounded-lg text-xs font-semibold">${p.category}</span></td>
                      <td class="p-4 font-bold text-slate-800">₱${p.price.toFixed(2)}</td>
                      <td class="p-4">
                        <span class="px-3 py-1 rounded-full text-xs font-semibold ${p.stock > 10 ? 'bg-emerald-50 text-emerald-700' : 'bg-red-50 text-red-700'}">
                          ${p.stock} ${p.unit} remaining
                        </span>
                      </td>
                      <td class="p-4 text-right space-x-2">
                        <button onclick="editProduct(${p.id})" class="px-3 py-1.5 bg-slate-100 hover:bg-slate-200 text-slate-700 rounded-lg text-xs font-semibold transition">Edit</button>
                        <button onclick="deleteProduct(${p.id})" class="px-3 py-1.5 bg-red-50 hover:bg-red-100 text-red-600 rounded-lg text-xs font-semibold transition">Delete</button>
                      </td>
                    </tr>
                  `).join('')}
                </tbody>
              </table>
            </div>
          </div>
        `;
      }

      function renderManagerEmployees() {
        return `
          <div class="max-w-5xl mx-auto space-y-6">
            <div class="flex justify-between items-center">
              <div>
                <h2 class="text-xl font-bold text-slate-900">Counter & Employee Accounts</h2>
                <p class="text-sm text-slate-500">Create and manage cashier accounts and staff privileges.</p>
              </div>
              <button onclick="showNotification('New employee account dialog opened!')" class="px-4 py-2 bg-indigo-600 hover:bg-indigo-700 text-white rounded-xl text-sm font-semibold shadow-md transition">
                <i class="fa-solid fa-user-plus mr-1.5"></i> Create Employee Account
              </button>
            </div>

            <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
              <div class="bg-white p-6 rounded-2xl shadow-sm border border-slate-200 flex items-center justify-between">
                <div>
                  <span class="text-xs font-bold uppercase tracking-wider px-2 py-0.5 bg-emerald-50 text-emerald-700 rounded-md">Cashier 1</span>
                  <h3 class="font-bold text-slate-900 mt-2">Counter Station 1</h3>
                  <p class="text-xs text-slate-500 mt-0.5">Username: cashier1@brew.com</p>
                </div>
                <span class="px-3 py-1 bg-emerald-50 text-emerald-700 rounded-full text-xs font-semibold">Active Shift</span>
              </div>
              <div class="bg-white p-6 rounded-2xl shadow-sm border border-slate-200 flex items-center justify-between">
                <div>
                  <span class="text-xs font-bold uppercase tracking-wider px-2 py-0.5 bg-indigo-50 text-indigo-700 rounded-md">Manager</span>
                  <h3 class="font-bold text-slate-900 mt-2">Main Branch Manager</h3>
                  <p class="text-xs text-slate-500 mt-0.5">Username: manager@pos.com</p>
                </div>
                <span class="px-3 py-1 bg-indigo-50 text-indigo-700 rounded-full text-xs font-semibold">Admin Access</span>
              </div>
            </div>
          </div>
        `;
      }

      function renderManagerVoidSetup() {
        return `
          <div class="max-w-4xl mx-auto space-y-6">
            <div>
              <h2 class="text-xl font-bold text-slate-900">Manager Access & Void Setup</h2>
              <p class="text-sm text-slate-500">Configure manager verification codes required for item voids, transaction voids, and special discounts.</p>
            </div>

            <div class="bg-white rounded-2xl p-8 shadow-sm border border-slate-200 space-y-6">
              <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
                <div>
                  <label class="block text-xs font-bold uppercase tracking-wider text-slate-600 mb-2">Manager Security PIN Code</label>
                  <input type="text" id="manager-pin-input" value="${state.managerCode}" class="w-full px-4 py-2.5 bg-slate-50 border border-slate-300 rounded-xl font-mono text-lg font-bold tracking-widest text-indigo-600 focus:ring-2 focus:ring-indigo-500 focus:outline-none">
                  <p class="text-xs text-slate-500 mt-1">Required at counter when voiding items, cancelling orders, or applying manager discounts.</p>
                </div>
                <div class="flex items-end">
                  <button onclick="updateManagerPIN()" class="w-full py-3 bg-indigo-600 hover:bg-indigo-700 text-white font-semibold rounded-xl text-sm transition shadow-md">
                    Save Manager PIN Code
                  </button>
                </div>
              </div>

              <hr class="border-slate-100 my-4">

              <div class="space-y-3">
                <h4 class="font-bold text-slate-800">Enforced Manager Authorizations at Counter:</h4>
                <div class="flex items-center justify-between p-3 bg-slate-50 rounded-xl">
                  <span class="text-sm font-medium text-slate-700">Item Void (Cancel specific item in cart)</span>
                  <span class="px-2.5 py-1 bg-indigo-100 text-indigo-800 rounded-lg text-xs font-bold">Requires Manager PIN</span>
                </div>
                <div class="flex items-center justify-between p-3 bg-slate-50 rounded-xl">
                  <span class="text-sm font-medium text-slate-700">Transaction / Order Void</span>
                  <span class="px-2.5 py-1 bg-indigo-100 text-indigo-800 rounded-lg text-xs font-bold">Requires Manager PIN</span>
                </div>
                <div class="flex items-center justify-between p-3 bg-slate-50 rounded-xl">
                  <span class="text-sm font-medium text-slate-700">Cash Drawer Cash-In / Cash-Out</span>
                  <span class="px-2.5 py-1 bg-indigo-100 text-indigo-800 rounded-lg text-xs font-bold">Requires Manager PIN</span>
                </div>
              </div>
            </div>
          </div>
        `;
      }


      // --- 4. COUNTER DASHBOARD ---
      function renderCounterDashboard() {
        const categories = ['All', ...new Set(state.products.map(p => p.category))];
        const filteredProducts = state.activeCategory === 'All' 
          ? state.products 
          : state.products.filter(p => p.category === state.activeCategory);

        const subtotal = state.cart.reduce((sum, item) => sum + (item.price * item.qty), 0);
        const vat = subtotal * 0.12;
        const total = subtotal;

        return `
          <div class="flex-1 flex flex-col h-full overflow-hidden bg-slate-100">
            <!-- Top Navbar -->
            <header class="h-16 bg-slate-900 text-white px-6 flex items-center justify-between shadow-md shrink-0">
              <div class="flex items-center space-x-4">
                <div class="w-10 h-10 bg-emerald-600 rounded-xl flex items-center justify-center font-bold text-lg">
                  <i class="fa-solid fa-cash-register"></i>
                </div>
                <div>
                  <h1 class="font-bold text-sm">POS Counter Terminal</h1>
                  <span class="text-xs text-emerald-400">Brew & Bake Café (Station 1)</span>
                </div>
              </div>
              <div class="flex items-center space-x-3">
                <button onclick="openDrawerAction()" class="px-3 py-2 bg-slate-800 hover:bg-slate-700 text-xs font-semibold rounded-xl border border-slate-700 transition">
                  <i class="fa-solid fa-drawer mr-1.5 text-indigo-400"></i> Cash Drawer (₱${state.cashDrawer.balance.toLocaleString()})
                </button>
                <button onclick="switchView('company_dashboard')" class="px-3 py-2 bg-indigo-600 hover:bg-indigo-700 text-xs font-semibold rounded-xl transition shadow-sm">
                  <i class="fa-solid fa-arrow-left mr-1"></i> Manager Dashboard
                </button>
              </div>
            </header>

            <!-- Main POS Interface (Side by Side) -->
            <div class="flex-1 flex overflow-hidden p-4 gap-4">
              <!-- Left: Product Catalog & Buttons -->
              <div class="flex-1 flex flex-col bg-white rounded-2xl shadow-sm border border-slate-200 overflow-hidden">
                <!-- Categories Bar -->
                <div class="p-4 border-b border-slate-200 flex gap-2 overflow-x-auto shrink-0 bg-slate-50">
                  ${categories.map(cat => `
                    <button onclick="setCounterCategory('${cat}')" class="px-4 py-2 rounded-xl text-xs font-bold transition whitespace-nowrap ${state.activeCategory === cat ? 'bg-brand-600 text-white shadow-md' : 'bg-white text-slate-600 border border-slate-200 hover:bg-slate-100'}">
                      ${cat}
                    </button>
                  `).join('')}
                </div>

                <!-- Product Buttons Grid -->
                <div class="flex-1 overflow-y-auto p-4">
                  <div class="grid grid-cols-2 md:grid-cols-3 lg:grid-cols-4 gap-4">
                    ${filteredProducts.map(p => `
                      <button onclick="addToCart(${p.id})" class="bg-gradient-to-br from-white to-slate-50 hover:from-brand-50 hover:to-indigo-50 border border-slate-200 hover:border-brand-500 rounded-2xl p-4 text-left flex flex-col justify-between shadow-sm hover:shadow-md transition group h-36">
                        <div>
                          <div class="flex justify-between items-start">
                            <span class="text-[10px] font-bold uppercase tracking-wider px-2 py-0.5 bg-slate-100 group-hover:bg-brand-100 text-slate-600 group-hover:text-brand-700 rounded-md">${p.category}</span>
                            <span class="text-xs font-semibold text-slate-500">${p.stock} left</span>
                          </div>
                          <h4 class="font-bold text-slate-900 mt-2 text-sm leading-snug group-hover:text-brand-700 line-clamp-2">${p.name}</h4>
                        </div>
                        <div class="flex justify-between items-center mt-2">
                          <span class="font-extrabold text-brand-600 text-base">₱${p.price.toFixed(2)}</span>
                          <span class="w-8 h-8 rounded-xl bg-slate-100 group-hover:bg-brand-600 text-slate-600 group-hover:text-white flex items-center justify-center transition shadow-sm">
                            <i class="fa-solid fa-plus text-xs"></i>
                          </span>
                        </div>
                      </button>
                    `).join('')}
                  </div>
                </div>
              </div>

              <!-- Right: Interactive Cart & Checkout -->
              <div class="w-96 bg-white rounded-2xl shadow-sm border border-slate-200 flex flex-col shrink-0 overflow-hidden">
                <div class="p-4 border-b border-slate-200 flex justify-between items-center bg-slate-50 shrink-0">
                  <h3 class="font-bold text-slate-900 flex items-center gap-2"><i class="fa-solid fa-cart-shopping text-brand-600"></i> Current Order</h3>
                  <button onclick="clearCart()" class="text-xs text-red-600 hover:text-red-700 font-semibold">Clear Cart</button>
                </div>

                <!-- Cart Items List (Scrollable) -->
                <div class="flex-1 overflow-y-auto p-4 space-y-3">
                  ${state.cart.length === 0 ? `
                    <div class="h-full flex flex-col items-center justify-center text-center text-slate-400 py-12">
                      <i class="fa-solid fa-basket-shopping text-4xl mb-3 text-slate-300"></i>
                      <p class="text-sm font-medium">Cart is empty</p>
                      <p class="text-xs text-slate-400 mt-1">Click product buttons to add items</p>
                    </div>
                  ` : state.cart.map((item, idx) => `
                    <div class="bg-slate-50 border border-slate-200/80 rounded-xl p-3 flex items-center justify-between">
                      <div class="flex-1 pr-2">
                        <h4 class="font-bold text-slate-800 text-xs">${item.name}</h4>
                        <div class="text-xs text-brand-600 font-bold mt-0.5">₱${item.price.toFixed(2)} x ${item.qty}</div>
                      </div>
                      <div class="flex items-center space-x-2">
                        <div class="flex items-center bg-white border border-slate-200 rounded-lg overflow-hidden shadow-sm">
                          <button onclick="updateCartQty(${idx}, -1)" class="px-2 py-1 hover:bg-slate-100 text-slate-600 text-xs"><i class="fa-solid fa-minus"></i></button>
                          <span class="px-2 text-xs font-bold text-slate-800">${item.qty}</span>
                          <button onclick="updateCartQty(${idx}, 1)" class="px-2 py-1 hover:bg-slate-100 text-slate-600 text-xs"><i class="fa-solid fa-plus"></i></button>
                        </div>
                        <button onclick="promptVoidItem(${idx})" class="text-red-500 hover:text-red-700 p-1 text-xs" title="Void Item (Requires Manager PIN)"><i class="fa-solid fa-trash"></i></button>
                      </div>
                    </div>
                  `).join('')}
                </div>

                <!-- Cart Summary & Checkout Actions -->
                <div class="p-4 border-t border-slate-200 bg-slate-50 space-y-3 shrink-0">
                  <div class="space-y-1 text-xs text-slate-600">
                    <div class="flex justify-between"><span>Subtotal</span> <span class="font-semibold text-slate-800">₱${subtotal.toFixed(2)}</span></div>
                    <div class="flex justify-between"><span>VAT (12% Included)</span> <span class="font-semibold text-slate-800">₱${vat.toFixed(2)}</span></div>
                    <div class="flex justify-between text-sm font-extrabold text-slate-900 pt-2 border-t border-slate-200">
                      <span>Total Amount</span>
                      <span class="text-brand-600 text-base">₱${total.toFixed(2)}</span>
                    </div>
                  </div>

                  <div class="grid grid-cols-2 gap-2 pt-1">
                    <button onclick="promptVoidTransaction()" class="py-2.5 bg-red-50 hover:bg-red-100 text-red-600 rounded-xl text-xs font-bold transition border border-red-200">
                      <i class="fa-solid fa-ban mr-1"></i> Void Order
                    </button>
                    <button onclick="processCheckout()" ${state.cart.length === 0 ? 'disabled' : ''} class="py-2.5 bg-brand-600 hover:bg-brand-700 disabled:opacity-50 text-white rounded-xl text-xs font-bold transition shadow-md shadow-brand-500/20">
                      <i class="fa-solid fa-credit-card mr-1"></i> Charge Total
                    </button>
                  </div>
                </div>
              </div>
            </div>
          </div>
        `;
      }


      // --- MODAL DIALOGS ENGINE ---
      function openEditSubscriptionModal(id) {
        const sub = state.subscriptions.find(s => s.id === id);
        state.modal = { type: 'edit_subscription', data: sub };
        renderModal();
      }

      function openAddProductModal() {
        state.modal = { type: 'add_product', data: null };
        renderModal();
      }

      function promptVoidItem(index) {
        state.modal = { type: 'manager_pin_auth', action: 'void_item', index };
        renderModal();
      }

      function promptVoidTransaction() {
        if (state.cart.length === 0) {
          showNotification('Cart is already empty', 'error');
          return;
        }
        state.modal = { type: 'manager_pin_auth', action: 'void_transaction' };
        renderModal();
      }

      function openDrawerAction() {
        state.modal = { type: 'manager_pin_auth', action: 'cash_drawer' };
        renderModal();
      }

      function renderModal() {
        // Remove existing modal if any
        const existing = document.getElementById('modal-overlay');
        if (existing) existing.remove();

        if (!state.modal.type) return;

        let modalContent = '';
        if (state.modal.type === 'edit_subscription') {
          const sub = state.modal.data;
          modalContent = `
            <div class="bg-white rounded-2xl max-w-md w-full p-6 shadow-2xl border border-slate-200 space-y-4">
              <h3 class="text-lg font-bold text-slate-900">Edit Subscription Pricing</h3>
              <p class="text-xs text-slate-500">Update pricing amount for <span class="font-semibold">${sub.name}</span>.</p>
              <div>
                <label class="block text-xs font-bold uppercase tracking-wider text-slate-600 mb-1">Pricing Amount (₱)</label>
                <input type="number" id="modal-sub-price" value="${sub.price}" class="w-full px-4 py-2.5 bg-slate-50 border border-slate-300 rounded-xl text-sm font-bold focus:ring-2 focus:ring-brand-500 focus:outline-none">
              </div>
              <div class="flex space-x-3 pt-2">
                <button onclick="closeModal()" class="flex-1 py-2.5 bg-slate-100 hover:bg-slate-200 text-slate-700 rounded-xl text-sm font-semibold transition">Cancel</button>
                <button onclick="saveSubscriptionPrice(${sub.id})" class="flex-1 py-2.5 bg-brand-600 hover:bg-brand-700 text-white rounded-xl text-sm font-semibold shadow-md transition">Save Price</button>
              </div>
            </div>
          `;
        } else if (state.modal.type === 'add_product') {
          modalContent = `
            <div class="bg-white rounded-2xl max-w-md w-full p-6 shadow-2xl border border-slate-200 space-y-4">
              <h3 class="text-lg font-bold text-slate-900">Add New Product</h3>
              <div class="space-y-3">
                <div>
                  <label class="block text-xs font-bold uppercase tracking-wider text-slate-600 mb-1">Product Name</label>
                  <input type="text" id="new-prod-name" placeholder="e.g. Matcha Latte" class="w-full px-4 py-2 bg-slate-50 border border-slate-300 rounded-xl text-sm">
                </div>
                <div>
                  <label class="block text-xs font-bold uppercase tracking-wider text-slate-600 mb-1">Category</label>
                  <input type="text" id="new-prod-cat" placeholder="e.g. Coffee" class="w-full px-4 py-2 bg-slate-50 border border-slate-300 rounded-xl text-sm">
                </div>
                <div>
                  <label class="block text-xs font-bold uppercase tracking-wider text-slate-600 mb-1">Price (₱)</label>
                  <input type="number" id="new-prod-price" placeholder="150" class="w-full px-4 py-2 bg-slate-50 border border-slate-300 rounded-xl text-sm">
                </div>
                <div>
                  <label class="block text-xs font-bold uppercase tracking-wider text-slate-600 mb-1">Initial Stock</label>
                  <input type="number" id="new-prod-stock" placeholder="50" class="w-full px-4 py-2 bg-slate-50 border border-slate-300 rounded-xl text-sm">
                </div>
              </div>
              <div class="flex space-x-3 pt-2">
                <button onclick="closeModal()" class="flex-1 py-2.5 bg-slate-100 hover:bg-slate-200 text-slate-700 rounded-xl text-sm font-semibold transition">Cancel</button>
                <button onclick="confirmAddProduct()" class="flex-1 py-2.5 bg-indigo-600 hover:bg-indigo-700 text-white rounded-xl text-sm font-semibold shadow-md transition">Add Product</button>
              </div>
            </div>
          `;
        } else if (state.modal.type === 'manager_pin_auth') {
          modalContent = `
            <div class="bg-white rounded-2xl max-w-sm w-full p-6 shadow-2xl border border-slate-200 space-y-4 text-center">
              <div class="w-12 h-12 bg-indigo-50 text-indigo-600 rounded-2xl mx-auto flex items-center justify-center text-xl font-bold">
                <i class="fa-solid fa-lock-keyhole"></i>
              </div>
              <h3 class="text-lg font-bold text-slate-900">Manager Authorization Required</h3>
              <p class="text-xs text-slate-500">Enter the 4-digit Manager PIN code to authorize this action.</p>
              <div>
                <input type="password" id="entered-manager-pin" maxlength="4" placeholder="••••" class="w-48 mx-auto px-4 py-2.5 bg-slate-50 border border-slate-300 rounded-xl text-center text-xl font-mono tracking-widest focus:ring-2 focus:ring-indigo-500 focus:outline-none">
              </div>
              <div class="flex space-x-3 pt-2">
                <button onclick="closeModal()" class="flex-1 py-2.5 bg-slate-100 hover:bg-slate-200 text-slate-700 rounded-xl text-sm font-semibold transition">Cancel</button>
                <button onclick="verifyManagerPIN()" class="flex-1 py-2.5 bg-indigo-600 hover:bg-indigo-700 text-white rounded-xl text-sm font-semibold shadow-md transition">Authorize</button>
              </div>
            </div>
          `;
        }

        const overlay = document.createElement('div');
        overlay.id = 'modal-overlay';
        overlay.className = 'fixed inset-0 z-50 bg-slate-900/60 backdrop-blur-sm flex items-center justify-center p-4 animate-fade-in';
        overlay.innerHTML = modalContent;
        document.body.appendChild(overlay);
      }

      function closeModal() {
        state.modal = { type: null, data: null };
        const overlay = document.getElementById('modal-overlay');
        if (overlay) overlay.remove();
        render();
      }


      // --- EVENT HANDLERS & ACTIONS ---
      function attachEventListeners() {
        const loginForm = document.getElementById('login-form');
        if (loginForm) {
          loginForm.onsubmit = (e) => {
            e.preventDefault();
            const email = document.getElementById('login-email').value;
            const pass = document.getElementById('login-pass').value;

            if (email === state.adminCredentials.email && pass === state.adminCredentials.pass) {
              state.currentView = 'admin_dashboard';
              showNotification('Welcome Super Admin!');
              render();
            } else if (email === state.companyManagerAuth.email || pass) {
              state.currentView = 'company_dashboard';
              showNotification('Welcome Company Manager!');
              render();
            } else {
              showNotification('Invalid credentials. Please check email and password.', 'error');
            }
          };
        }
      }

      function fillAdminCreds() {
        document.getElementById('login-email').value = state.adminCredentials.email;
        document.getElementById('login-pass').value = state.adminCredentials.pass;
      }

      function fillCompanyCreds() {
        document.getElementById('login-email').value = state.companyManagerAuth.email;
        document.getElementById('login-pass').value = state.companyManagerAuth.pass;
      }

      function switchView(view) {
        state.currentView = view;
        render();
      }

      function setAdminTab(tab) {
        adminTab = tab;
        render();
      }

      function setManagerTab(tab) {
        managerTab = tab;
        render();
      }

      function setCounterCategory(cat) {
        state.activeCategory = cat;
        render();
      }

      function logout() {
        state.currentView = 'login';
        state.cart = [];
        showNotification('Signed out successfully.');
        render();
      }

      // Admin Actions
      function saveSubscriptionPrice(id) {
        const newPrice = parseFloat(document.getElementById('modal-sub-price').value);
        const sub = state.subscriptions.find(s => s.id === id);
        if (sub && !isNaN(newPrice)) {
          sub.price = newPrice;
          showNotification(`Successfully updated ${sub.name} price to ₱${newPrice.toLocaleString()}`);
          closeModal();
        }
      }

      function toggleCompanyStatus(id) {
        const c = state.companies.find(comp => comp.id === id);
        if (c) {
          c.status = c.status === 'Suspended' ? 'Approved' : 'Suspended';
          showNotification(`Company ${c.name} status updated to ${c.status}`);
          render();
        }
      }

      function approveCompany(id) {
        const c = state.companies.find(comp => comp.id === id);
        if (c) {
          c.status = 'Approved';
          showNotification(`Company ${c.name} approved successfully! 14-Day Free Trial activated.`);
          render();
        }
      }

      // Manager Actions
      function updateManagerPIN() {
        const newPin = document.getElementById('manager-pin-input').value;
        if (newPin.length === 4) {
          state.managerCode = newPin;
          showNotification('Manager PIN updated successfully!');
        } else {
          showNotification('PIN code must be exactly 4 digits.', 'error');
        }
      }

      function openAddProductModal() {
        state.modal = { type: 'add_product' };
        renderModal();
      }

      function confirmAddProduct() {
        const name = document.getElementById('new-prod-name').value;
        const category = document.getElementById('new-prod-cat').value;
        const price = parseFloat(document.getElementById('new-prod-price').value);
        const stock = parseInt(document.getElementById('new-prod-stock').value);

        if (name && category && !isNaN(price) && !isNaN(stock)) {
          state.products.push({ id: state.products.length + 1, name, category, price, stock, unit: 'PCS' });
          showNotification(`Product "${name}" added successfully!`);
          closeModal();
        } else {
          showNotification('Please fill in all product fields correctly.', 'error');
        }
      }

      function editProduct(id) {
        showNotification('Product edit modal opened.');
      }

      function deleteProduct(id) {
        state.products = state.products.filter(p => p.id !== id);
        showNotification('Product deleted successfully.');
        render();
      }

      // POS Counter Actions
      function addToCart(id) {
        const product = state.products.find(p => p.id === id);
        if (!product) return;

        if (product.stock <= 0) {
          showNotification('Product is out of stock!', 'error');
          return;
        }

        const existing = state.cart.find(item => item.id === id);
        if (existing) {
          if (existing.qty < product.stock) {
            existing.qty++;
          } else {
            showNotification('Reached maximum stock limit.', 'error');
            return;
          }
        } else {
          state.cart.push({ id: product.id, name: product.name, price: product.price, qty: 1 });
        }
        render();
      }

      function updateCartQty(idx, change) {
        const item = state.cart[idx];
        const product = state.products.find(p => p.id === item.id);
        item.qty += change;
        if (item.qty <= 0) {
          state.cart.splice(idx, 1);
        } else if (product && item.qty > product.stock) {
          item.qty = product.stock;
          showNotification('Exceeds available stock quantity.', 'error');
        }
        render();
      }

      function clearCart() {
        state.cart = [];
        showNotification('Cart cleared.');
        render();
      }

      function verifyManagerPIN() {
        const pin = document.getElementById('entered-manager-pin').value;
        if (pin === state.managerCode) {
          const action = state.modal.action;
          closeModal();

          if (action === 'void_item') {
            const idx = state.modal.index;
            state.cart.splice(idx, 1);
            showNotification('Item voided successfully by manager authorization.');
          } else if (action === 'void_transaction') {
            state.cart = [];
            showNotification('Transaction voided successfully by manager authorization.');
          } else if (action === 'cash_drawer') {
            state.cashDrawer.balance += 1000;
            showNotification('Cash drawer movement authorized (+₱1,000.00 cash-in).');
          }
          render();
        } else {
          showNotification('Invalid Manager PIN code!', 'error');
        }
      }

      function processCheckout() {
        if (state.cart.length === 0) return;

        // Auto deduct inventory
        state.cart.forEach(cartItem => {
          const prod = state.products.find(p => p.id === cartItem.id);
          if (prod) {
            prod.stock -= cartItem.qty;
          }
        });

        const totalAmt = state.cart.reduce((sum, item) => sum + (item.price * item.qty), 0);
        state.salesHistory.unshift({
          id: 'INV-' + Math.floor(1000 + Math.random() * 9000),
          cashier: 'Counter 1',
          total: totalAmt,
          items: state.cart.reduce((sum, item) => sum + item.qty, 0),
          time: new Date().toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' }),
          status: 'Completed'
        });

        state.cart = [];
        showNotification(`Payment successful! Inventory auto-deducted.`);
        render();
      }

      // Initial Render
      render();
    </script>
  </body>
  </html>
</application>


