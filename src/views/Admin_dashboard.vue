<template>
    <div class="wrapper">
      <!-- Sidebar Component -->
      <Sidebar 
        @toggle-sidebar="toggleSidebar" 
        @load-page="loadPage"
        @logout="logout"
      />
  
      <!-- Main Content Area -->
      <div class="main" id="container">
        <main class="content px-3 py-4">
          <div class="container-fluid">
            <div class="mb-3">
              <div class="d-flex justify-content-between align-items-center">
                <h3 class="fw-bold fs-4 mb-3">Admin Dashboard</h3>
                
                <!-- User Profile Dropdown -->
                <div class="dropdown">
                    <a class="nav-link dropdown-toggle" href="#" id="userDropdown" role="button" data-bs-toggle="dropdown" aria-expanded="false">
                        <span class="mr-2 d-none d-lg-inline text-gray-600 small">{{ userName }}</span>
                    </a>
                    <ul class="dropdown-menu dropdown-menu-end" aria-labelledby="userDropdown">
                        <li><a class="dropdown-item" href="#">My Profile</a></li>
                        <li><a class="dropdown-item" href="#">Settings</a></li>
                        <li><hr class="dropdown-divider" /></li>
                        <li><a class="dropdown-item" href="#" @click="logout">Logout</a></li>
                    </ul>
                </div>

              </div>
  
              <div class="row">
                <!-- Stats Cards -->
                <div class="col-12 col-md-3 mb-4" v-for="(card, index) in stats" :key="index">
                  <div class="card border-0 shadow">
                    <div class="card-body py-4">
                      <h5 class="mb-2 fw-bold">{{ card.title }}</h5>
                      <p class="mb-2 fw-bold" :id="card.id">{{ card.value }}</p>
                    </div>
                  </div>
                </div>
              </div>
  
              <!-- Chart Containers -->
              <div class="row">
                <!-- Sales Chart -->
                <div class="col-md-8 mb-4">
                  <div class="card border-0 shadow">
                    <div class="card-body">
                      <h5 class="card-title">Sales Chart</h5>
                      <canvas id="salesChart" width="400" height="140"></canvas>
                    </div>
                  </div>
                </div>
                
                <!-- Order Status Doughnut Chart -->
                <div class="col-md-4 mb-4">
                  <div class="card border-0 shadow">
                    <div class="card-body">
                      <h5 class="card-title">Order Status Distribution</h5>
                      <div style="width: 300px; min-width: 200px; height: 300px;">
                        <canvas id="orderStatusDoughnutChart"></canvas>
                      </div>
                    </div>
                  </div>
                </div>
              </div>
  
            </div>
          </div>
        </main>
      </div>
    </div>
  </template>
  
  <script>
  import { ref, onMounted } from 'vue';
  import axios from 'axios';
  import Sidebar from '@/components/Sidebar.vue';
  import Chart from 'chart.js/auto';
  
  export default {
    components: {
      Sidebar,
      data() {
        return {
        stats,
        userName: '',
        loadPage,
        logout,
        userEmail: '',
      };
      },
      created() {
      // Retrieve user data from localStorage when the component is created
      this.userName = localStorage.getItem('user_name');
      this.userEmail = localStorage.getItem('user_email');
    },
    },
    setup() {
      const stats = ref([
        { title: 'Total Earnings', value: '$0.00', id: 'TotalEarn' },
        { title: 'Purchases', value: '0', id: 'Purchases' },
        { title: 'Customers', value: '0', id: 'Customers' },
        { title: 'Pending Orders', value: '0', id: 'PendingOrders' },
      ]);
      
  
      const loadPage = (page) => {
        document.getElementById('container').innerHTML = `<object type="text/html" data="${page}" style="width:100%; height:100vh;"></object>`;
      };
  
      const logout = () => {
        axios.post('/logout')
          .then(() => {
            window.location.href = '/login';
          })
          .catch((error) => {
            console.error('Logout failed:', error);
          });
      };
  
      const fetchStats = async () => {
        try {
          const response = await axios.get('/api/admin/stats');
          stats.value = response.data.stats;
        } catch (error) {
          console.error('Error fetching stats:', error);
        }
      };
  
      // Initialize the charts
      const initializeCharts = () => {
        // Sales Chart
        const ctxSales = document.getElementById('salesChart').getContext('2d');
        new Chart(ctxSales, {
          type: 'line',
          data: {
            labels: ['January', 'February', 'March', 'April'],
            datasets: [{
              label: 'Total Sales',
              data: [1000, 1200, 1500, 1700],
              backgroundColor: 'rgba(75, 192, 192, 0.2)',
              borderColor: 'rgba(75, 192, 192, 1)',
              borderWidth: 1,
              fill: true,
            }],
          },
        });
  
        // Order Status Doughnut Chart
        const ctxDoughnut = document.getElementById('orderStatusDoughnutChart').getContext('2d');
        new Chart(ctxDoughnut, {
          type: 'doughnut',
          data: {
            labels: ['Pending', 'Completed', 'Cancelled'],
            datasets: [{
              data: [5, 10, 2],
              backgroundColor: ['#FF6384', '#36A2EB', '#FFCE56'],
            }],
          },
        });
      };
  
      onMounted(() => {
        fetchStats();
        initializeCharts();
      });
  
      
    },
  };
  </script>
  
  <style scoped>
    @import '@/assets/css/adminstyle.css';  /* Using alias @ to point to the src folder */
  </style>
  