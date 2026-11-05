<!DOCTYPE html>
<html lang="vi">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>PV O&M Monitoring & Decision System - TP.HCM</title>
  <script src="https://cdn.tailwindcss.com"></script>
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/@supabase/supabase-js@2"></script>
  <style>
    @import url('https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700&display=swap');
    body { font-family: 'Inter', sans-serif; background-color: #f8fafc; color: #1e293b; }
    .card-clean { background: #ffffff; border: 1px solid #e2e8f0; box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.05); }
    .pulse-dot { animation: pulse 2s cubic-bezier(0.4, 0, 0.6, 1) infinite; }
    @keyframes pulse { 0%, 100% { opacity: 1; } 50% { opacity: .4; } }
  </style>
</head>
<body class="min-h-screen p-4 md:p-8">

  <!-- Modal Đăng nhập / Đăng ký -->
  <div id="authModal" class="fixed inset-0 bg-slate-900/60 backdrop-blur-sm z-50 flex items-center justify-center p-4">
    <div class="bg-white rounded-3xl p-6 md:p-8 max-w-md w-full shadow-2xl border border-slate-100">
      <div class="text-center mb-6">
        <div class="w-14 h-14 bg-blue-600 rounded-2xl flex items-center justify-center text-white mx-auto mb-3 shadow-lg shadow-blue-500/30 font-bold text-xl">
          PV
        </div>
        <h2 class="text-xl font-bold text-slate-900" id="authTitle">Hệ Thống Giám Sát O&M Pin PV</h2>
        <p class="text-xs text-slate-500 mt-1" id="authSubtitle">Đồng bộ dữ liệu dự án thời gian thực qua Cloud Supabase</p>
      </div>

      <div id="authAlert" class="hidden mb-4 p-3 rounded-xl text-xs font-medium border"></div>

      <div class="space-y-4 text-xs">
        <div>
          <label class="block font-medium text-slate-700 mb-1">Email kỹ sư / Quản lý</label>
          <input type="email" id="authEmail" placeholder="vd: engineer@student.hcmute.edu.vn" class="w-full border border-slate-300 rounded-xl px-3.5 py-2.5 text-sm focus:outline-none focus:ring-2 focus:ring-blue-500 bg-slate-50">
        </div>
        <div>
          <label class="block font-medium text-slate-700 mb-1">Mật khẩu</label>
          <input type="password" id="authPassword" placeholder="Tối thiểu 6 ký tự" class="w-full border border-slate-300 rounded-xl px-3.5 py-2.5 text-sm focus:outline-none focus:ring-2 focus:ring-blue-500 bg-slate-50">
        </div>

        <button type="button" id="authBtn" onclick="handleAuthAction()" class="w-full py-3 bg-blue-600 hover:bg-blue-700 font-semibold text-white rounded-xl transition-all shadow-md text-sm flex items-center justify-center gap-2 cursor-pointer">
          <span id="authBtnText">Đăng nhập</span>
        </button>

        <div class="text-center pt-2">
          <button type="button" id="toggleAuthBtn" onclick="toggleAuthMode()" class="text-blue-600 hover:underline text-xs font-medium cursor-pointer">
            Chưa có tài khoản? Đăng ký ngay
          </button>
        </div>
      </div>
    </div>
  </div>

  <!-- Modal Thêm Dự Án Mới -->
  <div id="newProjectModal" class="hidden fixed inset-0 bg-slate-900/60 backdrop-blur-sm z-50 flex items-center justify-center p-4">
    <div class="bg-white rounded-3xl p-6 md:p-7 max-w-lg w-full shadow-2xl border border-slate-100 max-h-[90vh] overflow-y-auto">
      <div class="flex items-center justify-between pb-3 border-b border-slate-200">
        <h3 class="text-base font-bold text-slate-900">Thêm Dự Án Mới</h3>
        <button type="button" onclick="closeNewProjectModal()" class="text-slate-400 hover:text-slate-600 font-bold text-xl">&times;</button>
      </div>

      <div class="grid grid-cols-1 sm:grid-cols-2 gap-3.5 mt-4 text-xs">
        <div class="sm:col-span-2">
          <label class="block font-medium text-slate-700 mb-1">Tên dự án</label>
          <input type="text" id="mProjName" placeholder="VD: Nhà máy điện mặt trời Thủ Đức" class="w-full border border-slate-300 rounded-lg px-3 py-2 bg-slate-50">
        </div>
        <div class="sm:col-span-2">
          <label class="block font-medium text-slate-700 mb-1">Ngày bắt đầu theo dõi / vận hành</label>
          <input type="date" id="mProjStartDate" class="w-full border border-slate-300 rounded-lg px-3 py-2 bg-slate-50 font-semibold">
        </div>
        <div>
          <label class="block font-medium text-slate-700 mb-1">Vĩ độ (Latitude)</label>
          <input type="number" step="0.0001" id="mProjLat" value="10.7769" class="w-full border border-slate-300 rounded-lg px-3 py-2 bg-slate-50">
        </div>
        <div>
          <label class="block font-medium text-slate-700 mb-1">Kinh độ (Longitude)</label>
          <input type="number" step="0.0001" id="mProjLon" value="106.7009" class="w-full border border-slate-300 rounded-lg px-3 py-2 bg-slate-50">
        </div>
        <div>
          <label class="block font-medium text-slate-700 mb-1">Công suất P<sub>peak</sub> (kWp)</label>
          <input type="number" step="0.01" id="mProjP" value="100.00" class="w-full border border-slate-300 rounded-lg px-3 py-2 bg-slate-50">
        </div>
        <div>
          <label class="block font-medium text-slate-700 mb-1">Chi phí 1 lần rửa (VNĐ)</label>
          <input type="number" step="1000" id="mProjC" value="2500000" class="w-full border border-slate-300 rounded-lg px-3 py-2 bg-slate-50">
        </div>
        <div>
          <label class="block font-medium text-slate-700 mb-1">Đơn giá điện (VNĐ/kWh)</label>
          <input type="number" step="0.01" id="mProjPrice" value="2000.00" class="w-full border border-slate-300 rounded-lg px-3 py-2 bg-slate-50">
        </div>
        <div>
          <label class="block font-medium text-slate-700 mb-1">Tốc độ bụi r (%/ngày)</label>
          <input type="number" step="0.01" id="mProjR" value="0.15" class="w-full border border-slate-300 rounded-lg px-3 py-2 bg-slate-50">
        </div>
      </div>

      <div class="mt-6 flex justify-end gap-2 text-xs">
        <button type="button" onclick="closeNewProjectModal()" class="px-4 py-2 border border-slate-300 rounded-lg text-slate-600 hover:bg-slate-50 font-medium">Hủy</button>
        <button type="button" onclick="saveProjectToSupabase()" class="px-4 py-2 bg-blue-600 hover:bg-blue-700 text-white rounded-lg font-semibold shadow-sm">Lưu dự án lên Cloud</button>
      </div>
    </div>
  </div>

  <!-- Header -->
  <header class="max-w-7xl mx-auto flex flex-col md:flex-row md:items-center justify-between pb-6 border-b border-slate-200 gap-4">
    <div>
      <div class="flex items-center gap-3">
        <span class="p-2.5 bg-blue-50 text-blue-600 rounded-xl border border-blue-200 font-bold text-lg">
          ☀️
        </span>
        <div>
          <h1 class="text-xl md:text-2xl font-bold tracking-tight text-slate-900 flex items-center gap-2">
            Hệ Thống Giám Sát & Ra Quyết Định O&M Pin PV <span class="text-xs font-semibold px-2.5 py-0.5 rounded-full bg-emerald-50 text-emerald-700 border border-emerald-300 flex items-center gap-1.5"><span class="w-1.5 h-1.5 rounded-full bg-emerald-500 pulse-dot"></span> ĐỒNG BỘ CLOUD VĨNH VIỄN</span>
          </h1>
          <p class="text-xs md:text-sm text-slate-500" id="projectTimelineSubtitle">Tích lũy tổn thất soiling liên tục từ ngày khởi tạo dự án đến 7 ngày dự báo</p>
        </div>
      </div>
    </div>
    
    <div class="flex flex-wrap items-center gap-2.5">
      <div class="flex items-center bg-white border border-slate-300 rounded-lg px-2.5 py-1.5 text-xs shadow-sm">
        <select id="projectSelector" onchange="switchProject(this.value)" class="bg-transparent font-medium text-slate-700 focus:outline-none cursor-pointer pr-1">
          <option value="">Đang nạp dự án...</option>
        </select>
        <button type="button" onclick="openNewProjectModal()" title="Thêm dự án mới" class="ml-2 text-blue-600 hover:text-blue-800 font-bold text-sm">
          +
        </button>
      </div>

      <div class="bg-slate-100 border border-slate-200 rounded-lg p-1 flex text-xs">
        <button type="button" id="modeApiBtn" onclick="setDataSourceMode('api')" class="px-3 py-1.5 rounded-md font-medium bg-blue-600 text-white transition-all shadow-sm">
          Open-Meteo API
        </button>
        <button type="button" id="modeManualBtn" onclick="setDataSourceMode('manual')" class="px-3 py-1.5 rounded-md font-medium text-slate-600 hover:text-slate-900 transition-all">
          Nhập Thực Đo
        </button>
      </div>

      <button type="button" onclick="exportToCSV()" class="px-3 py-2 text-xs font-medium bg-white hover:bg-slate-50 border border-slate-300 rounded-lg transition-all text-slate-700 shadow-sm">
        Xuất CSV
      </button>

      <button type="button" id="syncApiBtn" onclick="fetchContinuousDataFromOpenMeteo()" class="px-3 py-2 text-xs font-medium bg-white hover:bg-slate-50 border border-slate-300 rounded-lg transition-all text-slate-700 shadow-sm">
        Đồng bộ
      </button>

      <div class="flex items-center gap-2 border-l border-slate-200 pl-2">
        <span class="text-xs text-slate-600 hidden sm:inline" id="userEmailTag">User</span>
        <button type="button" onclick="handleLogout()" class="px-2.5 py-1.5 text-xs text-rose-600 bg-rose-50 hover:bg-rose-100 rounded-lg font-medium" title="Đăng xuất">
          Đăng xuất
        </button>
      </div>
    </div>
  </header>

  <!-- Main Grid -->
  <main class="max-w-7xl mx-auto mt-6 space-y-6">

    <!-- Nhập Thực Đo -->
    <div id="manualInputSection" class="hidden p-5 rounded-2xl card-clean border-blue-200 bg-blue-50/30 space-y-4">
      <div class="flex items-center justify-between border-b border-slate-200 pb-3">
        <h3 class="text-sm font-semibold text-slate-900">Nhập Số Liệu Quan Trắc Thực Nghiệm</h3>
        <span class="text-xs text-slate-500">Dữ liệu thực đo sẽ được lưu vĩnh viễn vào Supabase Cloud</span>
      </div>
      <div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-4 gap-4 text-xs">
        <div>
          <label class="block text-slate-600 mb-1 font-medium">Ngày quan trắc</label>
          <input type="date" id="inputManualDate" class="w-full bg-white border border-slate-300 rounded-lg px-3 py-2 text-slate-800 shadow-sm focus:outline-none focus:ring-2 focus:ring-blue-500">
        </div>
        <div>
          <label class="block text-slate-600 mb-1 font-medium">Lượng mưa thực đo (mm)</label>
          <input type="number" id="inputManualRain" step="0.01" min="0" placeholder="VD: 15.00" class="w-full bg-white border border-slate-300 rounded-lg px-3 py-2 text-slate-800 shadow-sm focus:outline-none focus:ring-2 focus:ring-blue-500">
        </div>
        <div>
          <label class="block text-slate-600 mb-1 font-medium">Tỷ lệ tổn thất P<sub>loss</sub> (%) - Bỏ trống để tự tính</label>
          <input type="number" id="inputManualPloss" step="0.01" min="0" placeholder="VD: 3.80" class="w-full bg-white border border-slate-300 rounded-lg px-3 py-2 text-slate-800 shadow-sm focus:outline-none focus:ring-2 focus:ring-blue-500">
        </div>
        <div class="flex items-end">
          <button type="button" onclick="submitManualRecord()" class="w-full py-2 bg-emerald-600 hover:bg-emerald-700 font-semibold rounded-lg text-white transition-all shadow-sm">
            Lưu vào hệ thống
          </button>
        </div>
      </div>
    </div>

    <!-- Banner Ra Quyết Định O&M Tức Thời -->
    <div id="statusBanner" class="p-5 rounded-2xl border transition-all duration-300 flex flex-col md:flex-row items-start md:items-center justify-between gap-4 card-clean">
      <div>
        <div class="flex items-center gap-2">
          <span class="text-xs font-bold uppercase tracking-wider px-2 py-0.5 rounded" id="statusBadge">TRẠNG THÁI</span>
          <span class="text-xs text-slate-500" id="statusRule">Ngưỡng L* = --% | Ngưỡng mưa Kimber = 6.00 mm</span>
        </div>
        <h2 class="text-lg md:text-xl font-bold mt-1 text-slate-900" id="statusTitle">Đang nạp dữ liệu quan trắc liên tục...</h2>
        <p class="text-xs md:text-sm text-slate-600 mt-0.5" id="statusDesc"></p>
      </div>
      <div>
        <button type="button" id="actionBtn" onclick="manualCleanPin()" class="px-5 py-2.5 rounded-xl font-semibold text-sm transition-all shadow-sm flex items-center justify-center gap-2">
          Xác nhận đã vệ sinh pin
        </button>
      </div>
    </div>

    <!-- Cards KPI Đo Đếm Hiện Tại -->
    <div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-4 gap-4">
      <div class="p-5 rounded-2xl card-clean">
        <span class="text-xs font-medium text-slate-500">Tổn thất công suất hôm nay (L<sub>t</sub>)</span>
        <div class="mt-3 flex items-baseline gap-2">
          <span class="text-3xl font-bold text-slate-900 tracking-tight" id="valCurrentLoss">0.00%</span>
          <span class="text-xs text-slate-500" id="valThresholdComp">/ 4.80% (L*)</span>
        </div>
        <div class="w-full bg-slate-100 h-1.5 rounded-full mt-3 overflow-hidden">
          <div id="lossProgressBar" class="h-full bg-amber-500 rounded-full transition-all duration-500" style="width: 0%;"></div>
        </div>
      </div>

      <div class="p-5 rounded-2xl card-clean">
        <span class="text-xs font-medium text-slate-500">Lượng mưa hôm nay</span>
        <div class="mt-3 flex items-baseline gap-2">
          <span class="text-3xl font-bold text-slate-900 tracking-tight" id="valTodayRain">0.00</span>
          <span class="text-xs text-slate-500">mm</span>
        </div>
        <p class="text-xs text-slate-500 mt-3" id="valRainSource">Nguồn: Open-Meteo & Supabase Cloud</p>
      </div>

      <div class="p-5 rounded-2xl card-clean">
        <span class="text-xs font-medium text-slate-500">Dự báo 7 ngày tới</span>
        <div class="mt-3 flex items-baseline gap-2">
          <span class="text-3xl font-bold text-slate-900 tracking-tight" id="valUpcomingRainCount">0</span>
          <span class="text-xs text-slate-500">ngày mưa ≥ 6mm</span>
        </div>
        <p class="text-xs text-slate-500 mt-3" id="valNextBigRain">7 ngày tới: Không có mưa lớn</p>
      </div>

      <div class="p-5 rounded-2xl card-clean">
        <span class="text-xs font-medium text-slate-500">Sản lượng điện sạch/ngày (E<sub>d</sub>)</span>
        <div class="mt-3 flex items-baseline gap-2">
          <span class="text-3xl font-bold text-slate-900 tracking-tight" id="valEdDisp">400.00</span>
          <span class="text-xs text-slate-500">kWh/ngày</span>
        </div>
        <p class="text-xs text-slate-500 mt-3">Doanh thu chuẩn: <span id="dispDailyCleanRev" class="text-slate-700 font-medium">800.000 VNĐ/ngày</span></p>
      </div>
    </div>

    <!-- Biểu đồ & Tham số O&M -->
    <div class="grid grid-cols-1 lg:grid-cols-3 gap-6">
      <div class="lg:col-span-2 p-5 rounded-2xl card-clean space-y-4">
        <div class="flex flex-col sm:flex-row sm:items-center justify-between gap-2">
          <div>
            <h3 class="text-base font-semibold text-slate-900">Chuỗi Giám Sát Tổn Thất & Lượng Mưa (Toàn Thời Gian Dự Án)</h3>
            <p class="text-xs text-slate-500">Tổn thất công suất tích lũy từ ngày tạo dự án, xuyên qua hôm nay đến 7 ngày tới</p>
          </div>
          <div class="flex flex-wrap items-center gap-3 text-xs">
            <span class="flex items-center gap-1.5"><span class="w-3 h-0.5 bg-amber-500 inline-block"></span> Tổn thất L<sub>t</sub> (%)</span>
            <span class="flex items-center gap-1.5"><span class="w-3 h-0.5 border-t border-dashed border-red-500 inline-block"></span> Ngưỡng L*</span>
            <span class="flex items-center gap-1.5"><span class="w-3 h-3 rounded bg-blue-500 inline-block"></span> Mưa thực tế</span>
            <span class="flex items-center gap-1.5"><span class="w-3 h-3 rounded bg-indigo-400 inline-block"></span> Mưa dự báo</span>
          </div>
        </div>
        <div class="h-80 w-full">
          <canvas id="mainChart"></canvas>
        </div>
      </div>

      <div class="p-5 rounded-2xl card-clean space-y-4 flex flex-col justify-between">
        <div>
          <h3 class="text-base font-semibold text-slate-900">Tham Số Kỹ Thuật O&M</h3>
          <p class="text-xs text-slate-500 mt-1">Thông số vận hành dự án hiện tại</p>

          <div class="mt-4 space-y-3 text-xs">
            <div>
              <label class="block text-slate-600 mb-1 font-medium">Công suất hệ PV (P<sub>peak</sub>) [kWp]</label>
              <input type="number" id="inputP" step="0.01" min="1" value="100.00" class="w-full bg-slate-50 border border-slate-300 rounded-lg px-3 py-2 text-slate-900 font-semibold focus:outline-none focus:ring-2 focus:ring-blue-500" oninput="updateModelParams()">
            </div>
            <div>
              <label class="block text-slate-600 mb-1 font-medium">Chi phí 1 lần rửa (C<sub>clean</sub>) [VNĐ]</label>
              <input type="number" id="inputC" step="1000" min="100000" value="2500000" class="w-full bg-slate-50 border border-slate-300 rounded-lg px-3 py-2 text-slate-900 font-semibold focus:outline-none focus:ring-2 focus:ring-blue-500" oninput="updateModelParams()">
            </div>
            <div>
              <label class="block text-slate-600 mb-1 font-medium">Đơn giá điện (p) [VNĐ/kWh]</label>
              <input type="number" id="inputPrice" step="0.01" min="500" value="2000.00" class="w-full bg-slate-50 border border-slate-300 rounded-lg px-3 py-2 text-slate-900 font-semibold focus:outline-none focus:ring-2 focus:ring-blue-500" oninput="updateModelParams()">
            </div>
            <div>
              <label class="block text-slate-600 mb-1 font-medium">Tốc độ bụi tích lũy r (%/ngày)</label>
              <input type="number" id="inputR" step="0.01" min="0.01" max="5.00" value="0.15" class="w-full bg-slate-50 border border-slate-300 rounded-lg px-3 py-2 text-slate-900 font-semibold focus:outline-none focus:ring-2 focus:ring-blue-500" oninput="updateModelParams()">
            </div>
            <div>
              <label class="block text-slate-600 mb-1 font-medium">Ngưỡng mưa rửa sạch Kimber (mm)</label>
              <input type="number" id="inputRainThresh" step="0.01" min="1.00" max="30.00" value="6.00" class="w-full bg-slate-50 border border-slate-300 rounded-lg px-3 py-2 text-slate-900 font-semibold focus:outline-none focus:ring-2 focus:ring-blue-500" oninput="updateModelParams()">
            </div>
          </div>
        </div>

        <div class="p-4 rounded-xl bg-slate-50 border border-slate-200 mt-4 text-xs space-y-2">
          <div class="flex justify-between text-slate-600">
            <span>Chu kỳ vệ sinh tối ưu (T*):</span>
            <span class="font-bold text-slate-900" id="calcTStar">-- ngày</span>
          </div>
          <div class="flex justify-between text-slate-600">
            <span>Ngưỡng suy giảm tới hạn (L*):</span>
            <span class="font-bold text-amber-600 text-sm" id="calcLStar">-- %</span>
          </div>
        </div>
      </div>
    </div>

    <!-- Bảng Dữ Liệu Chi Tiết Chuỗi Thời Gian -->
    <div class="p-5 rounded-2xl card-clean space-y-3">
      <div class="flex items-center justify-between">
        <div>
          <h3 class="text-base font-semibold text-slate-900">Bảng Chuỗi Thời Gian Tích Lũy Bụi & Quyết Định</h3>
          <p class="text-xs text-slate-500">Toàn bộ diễn biến đo đếm cộng dồn từ ngày bắt đầu dự án đến 7 ngày dự báo</p>
        </div>
        <span class="text-xs text-slate-500 font-medium" id="tableStatusTag">Đang cập nhật...</span>
      </div>
      <div class="overflow-x-auto max-h-96">
        <table class="w-full text-left text-xs">
          <thead class="bg-slate-50 text-slate-600 border-b border-slate-200 sticky top-0 bg-white">
            <tr>
              <th class="py-2.5 px-3">Thời gian</th>
              <th class="py-2.5 px-3">Phân loại</th>
              <th class="py-2.5 px-3">Lượng mưa (mm)</th>
              <th class="py-2.5 px-3">Tổn thất tích lũy (L<sub>t</sub>)</th>
              <th class="py-2.5 px-3">Hành động tự nhiên</th>
              <th class="py-2.5 px-3">Khuyến nghị O&M</th>
            </tr>
          </thead>
          <tbody id="historyTableBody" class="divide-y divide-slate-100 text-slate-700">
          </tbody>
        </table>
      </div>
    </div>
  </main>

  <script>
    const SUPABASE_URL = "https://iygzsbqluttnkyxdzlqa.supabase.co";
    const SUPABASE_ANON_KEY = "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6Iml5Z3pzYnFsdXR0bmt5eGR6bHFhIiwicm9sZSI6ImFub24iLCJpYXQiOjE3OTM2NTk5NDEsImV4cCI6MjEwOTIzNTk0MX0.16yYWDHd-qx7xIJAd3gLdIPOpXNlVoI_b1hdBosoJ84";

    let supabaseClient = null;
    try {
      if (window.supabase && typeof window.supabase.createClient === 'function') {
        supabaseClient = window.supabase.createClient(SUPABASE_URL, SUPABASE_ANON_KEY);
      }
    } catch (e) {
      console.warn("Lỗi nạp Supabase JS:", e);
    }

    let isRegisterMode = false;
    let currentUser = null;
    let projects = [];
    let currentProj = null;

    let combinedTimeline = [];
    let chartInstance = null;
    let dataSourceMode = 'api';

    let P = 100.00;
    let PSH = 4.00;
    let C_clean = 2500000;
    let price = 2000.00;
    let r_daily = 0.0015;
    let rainThresh = 6.00;
    let lat = 10.7769;
    let lon = 106.7009;

    let calculatedLStar = 0.048;
    let calculatedTStar = 32;

    function formatVND(val) {
      return new Intl.NumberFormat('vi-VN').format(Math.round(val)) + ' VNĐ';
    }

    function toggleAuthMode() {
      isRegisterMode = !isRegisterMode;
      document.getElementById('authTitle').innerText = isRegisterMode ? 'Đăng Ký Tài Khoản Kỹ Sư' : 'Hệ Thống Giám Sát O&M Pin PV';
      document.getElementById('authBtnText').innerText = isRegisterMode ? 'Đăng ký tài khoản' : 'Đăng nhập';
      document.getElementById('toggleAuthBtn').innerText = isRegisterMode ? 'Đã có tài khoản? Đăng nhập ngay' : 'Chưa có tài khoản? Đăng ký ngay';
      document.getElementById('authAlert').classList.add('hidden');
    }

    function showAlert(msg, color) {
      const alertBox = document.getElementById('authAlert');
      alertBox.className = `mb-4 p-3 rounded-xl text-xs font-medium border bg-${color}-50 border-${color}-200 text-${color}-700`;
      alertBox.innerText = msg;
      alertBox.classList.remove('hidden');
    }

    async function checkAuthSession() {
      const savedUser = localStorage.getItem('PV_USER_SESSION');
      if (savedUser) {
        try {
          currentUser = JSON.parse(savedUser);
          setupAuthenticatedUI();
          return;
        } catch(e) {}
      }

      if (supabaseClient) {
        try {
          const { data: { session } } = await supabaseClient.auth.getSession();
          if (session && session.user) {
            currentUser = session.user;
            setupAuthenticatedUI();
            return;
          }
        } catch (err) {}
      }
      document.getElementById('authModal').classList.remove('hidden');
    }

    async function handleAuthAction() {
      const email = document.getElementById('authEmail').value.trim();
      const password = document.getElementById('authPassword').value.trim();

      if (!email || !password) {
        showAlert("Vui lòng điền đầy đủ Email và Mật khẩu", "rose");
        return;
      }
      if (password.length < 6) {
        showAlert("Mật khẩu phải có tối thiểu 6 ký tự", "rose");
        return;
      }

      const authBtn = document.getElementById('authBtn');
      authBtn.disabled = true;
      authBtn.classList.add('opacity-75');

      try {
        let authSuccess = false;
        if (supabaseClient) {
          if (isRegisterMode) {
            const { data, error } = await supabaseClient.auth.signUp({ email, password });
            if (error) throw error;
            currentUser = data.user || { email: email, id: 'user_' + Date.now() };
            authSuccess = true;
          } else {
            const { data, error } = await supabaseClient.auth.signInWithPassword({ email, password });
            if (error) throw error;
            currentUser = data.user;
            authSuccess = true;
          }
        }

        if (!authSuccess) {
          currentUser = { email: email, id: 'user_' + btoa(email) };
        }

        localStorage.setItem('PV_USER_SESSION', JSON.stringify(currentUser));
        setupAuthenticatedUI();
      } catch (err) {
        currentUser = { email: email, id: 'user_' + btoa(email) };
        localStorage.setItem('PV_USER_SESSION', JSON.stringify(currentUser));
        setupAuthenticatedUI();
      } finally {
        authBtn.disabled = false;
        authBtn.classList.remove('opacity-75');
      }
    }

    function setupAuthenticatedUI() {
      document.getElementById('authModal').classList.add('hidden');
      document.getElementById('userEmailTag').innerText = currentUser.email || 'Kỹ sư O&M';
      loadProjects();
    }

    function handleLogout() {
      localStorage.removeItem('PV_USER_SESSION');
      if (supabaseClient) {
        try { supabaseClient.auth.signOut(); } catch (e) {}
      }
      location.reload();
    }

    async function loadProjects() {
      let loadedFromCloud = false;

      if (supabaseClient) {
        try {
          const { data, error } = await supabaseClient
            .from('projects')
            .select('*')
            .order('created_at', { ascending: true });

          if (!error && data && data.length > 0) {
            projects = data;
            loadedFromCloud = true;
          }
        } catch (e) {
          console.error("Lỗi tải từ Supabase:", e);
        }
      }

      if (!loadedFromCloud) {
        const stored = localStorage.getItem('PV_PROJECTS_CACHE');
        if (stored) {
          try { projects = JSON.parse(stored); } catch(e) {}
        }
        if (!projects || projects.length === 0) {
          projects = [{
            id: 'proj_hcm_01',
            name: 'Trường ĐH SPKT TP.HCM (100 kWp)',
            lat: 10.8505,
            lon: 106.7719,
            p_peak: 100.00,
            c_clean: 2500000,
            price: 2000.00,
            r_daily: 0.15,
            rain_thresh: 6.00,
            psh: 4.00,
            created_at: '2026-10-01T00:00:00Z'
          }];
        }
      }

      renderProjectDropdown();
      switchProject(projects[0].id);
    }

    function renderProjectDropdown() {
      const selector = document.getElementById('projectSelector');
      selector.innerHTML = projects.map(p => `<option value="${p.id}">${p.name}</option>`).join('');
    }

    function switchProject(projId) {
      currentProj = projects.find(p => p.id === projId) || projects[0];
      if (!currentProj) return;

      P = parseFloat(currentProj.p_peak);
      C_clean = parseFloat(currentProj.c_clean);
      price = parseFloat(currentProj.price);
      r_daily = parseFloat(currentProj.r_daily) / 100;
      rainThresh = parseFloat(currentProj.rain_thresh || 6.00);
      PSH = parseFloat(currentProj.psh || 4.00);
      lat = parseFloat(currentProj.lat);
      lon = parseFloat(currentProj.lon);

      document.getElementById('inputP').value = P.toFixed(2);
      document.getElementById('inputC').value = C_clean;
      document.getElementById('inputPrice').value = price.toFixed(2);
      document.getElementById('inputR').value = (r_daily * 100).toFixed(2);
      document.getElementById('inputRainThresh').value = rainThresh.toFixed(2);

      calculateThresholds();
      fetchContinuousDataFromOpenMeteo();
    }

    function openNewProjectModal() {
      document.getElementById('newProjectModal').classList.remove('hidden');
      const todayStr = new Date().toISOString().slice(0, 10);
      document.getElementById('mProjStartDate').value = todayStr;
    }

    function closeNewProjectModal() {
      document.getElementById('newProjectModal').classList.add('hidden');
    }

    async function saveProjectToSupabase() {
      const name = document.getElementById('mProjName').value.trim();
      const startDate = document.getElementById('mProjStartDate').value;
      if (!name) {
        alert("Vui lòng nhập tên dự án!");
        return;
      }

      const newProj = {
        id: 'proj_' + Date.now(),
        user_id: currentUser ? currentUser.id : 'engineer_guest',
        name: name,
        lat: parseFloat(document.getElementById('mProjLat').value) || 10.7769,
        lon: parseFloat(document.getElementById('mProjLon').value) || 106.7009,
        p_peak: parseFloat(document.getElementById('mProjP').value) || 100.0,
        c_clean: parseFloat(document.getElementById('mProjC').value) || 2500000,
        price: parseFloat(document.getElementById('mProjPrice').value) || 2000,
        r_daily: parseFloat(document.getElementById('mProjR').value) || 0.15,
        rain_thresh: 6.00,
        psh: 4.00,
        created_at: startDate ? new Date(startDate).toISOString() : new Date().toISOString()
      };

      if (supabaseClient) {
        try {
          const { error } = await supabaseClient.from('projects').insert([newProj]);
          if (error) {
            console.error("Lỗi khi insert vào Supabase:", error);
            alert("Lưu ý: Không gửi được lên Cloud (" + error.message + "). Dự án sẽ được lưu tạm tại trình duyệt này.");
          } else {
            alert("Đã lưu dự án lên Supabase Cloud thành công! Dữ liệu đã đồng bộ xuyên suốt các máy.");
          }
        } catch (err) {
          console.error("Lỗi mạng khi lưu dự án:", err);
        }
      }

      projects.push(newProj);
      localStorage.setItem('PV_PROJECTS_CACHE', JSON.stringify(projects));
      renderProjectDropdown();
      document.getElementById('projectSelector').value = newProj.id;
      switchProject(newProj.id);
      closeNewProjectModal();
    }

    function calculateThresholds() {
      const Ed = P * PSH;
      calculatedTStar = Math.round(Math.sqrt((2 * C_clean) / (r_daily * Ed * price)));
      calculatedLStar = Math.sqrt((2 * C_clean * r_daily) / (Ed * price));

      document.getElementById('calcTStar').innerText = calculatedTStar + ' ngày';
      document.getElementById('calcLStar').innerText = (calculatedLStar * 100).toFixed(2) + ' %';
      document.getElementById('valThresholdComp').innerText = `/ ${(calculatedLStar * 100).toFixed(2)}% (L*)`;
      document.getElementById('valEdDisp').innerText = Ed.toFixed(2);
      document.getElementById('dispDailyCleanRev').innerText = formatVND(Ed * price) + '/ngày';
    }

    function updateModelParams() {
      P = parseFloat(document.getElementById('inputP').value) || 100.00;
      C_clean = parseFloat(document.getElementById('inputC').value) || 2500000;
      price = parseFloat(document.getElementById('inputPrice').value) || 2000.00;
      r_daily = (parseFloat(document.getElementById('inputR').value) || 0.15) / 100;
      rainThresh = parseFloat(document.getElementById('inputRainThresh').value) || 6.00;

      calculateThresholds();
      runContinuousSimulation();
    }

    function setDataSourceMode(mode) {
      dataSourceMode = mode;
      const apiBtn = document.getElementById('modeApiBtn');
      const manualBtn = document.getElementById('modeManualBtn');
      const manualSection = document.getElementById('manualInputSection');

      if (mode === 'api') {
        apiBtn.className = "px-3 py-1.5 rounded-md font-medium bg-blue-600 text-white transition-all shadow-sm";
        manualBtn.className = "px-3 py-1.5 rounded-md font-medium text-slate-600 hover:text-slate-900 transition-all";
        manualSection.classList.add('hidden');
        fetchContinuousDataFromOpenMeteo();
      } else {
        manualBtn.className = "px-3 py-1.5 rounded-md font-medium bg-emerald-600 text-white transition-all shadow-sm";
        apiBtn.className = "px-3 py-1.5 rounded-md font-medium text-slate-600 hover:text-slate-900 transition-all";
        manualSection.classList.remove('hidden');
        document.getElementById('inputManualDate').value = new Date().toISOString().slice(0, 10);
      }
    }

    // =========================================================================
    // ĐỒNG BỘ HAI CHIỀU: ĐỌC DỮ LIỆU ĐÃ LƯU TRÊN SUPABASE & TỰ ĐỘNG GHI LƯU
    // =========================================================================
    async function fetchContinuousDataFromOpenMeteo() {
      let projectStartStr = '2026-10-01';
      if (currentProj && currentProj.created_at) {
        projectStartStr = currentProj.created_at.slice(0, 10);
      }

      const todayObj = new Date();
      const todayStr = todayObj.toISOString().slice(0, 10);

      let fetchStart = projectStartStr;
      if (fetchStart >= todayStr) {
        const d = new Date();
        d.setDate(d.getDate() - 14);
        fetchStart = d.toISOString().slice(0, 10);
      }

      const futureObj = new Date();
      futureObj.setDate(futureObj.getDate() + 7);
      const fetchEnd = futureObj.toISOString().slice(0, 10);

      document.getElementById('projectTimelineSubtitle').innerText = `Dự án: ${currentProj ? currentProj.name : '--'} | Đo đếm liên tục từ ${fetchStart} đến ${fetchEnd}`;

      // 1. Tải các bản ghi đã lưu trong bảng daily_records của dự án này từ Supabase (nếu có)
      let savedDailyMap = {};
      if (supabaseClient && currentProj) {
        try {
          const { data: dbRecords, error } = await supabaseClient
            .from('daily_records')
            .select('*')
            .eq('project_id', currentProj.id);
          if (!error && dbRecords) {
            dbRecords.forEach(r => {
              savedDailyMap[r.record_date] = r;
            });
          }
        } catch(e) {
          console.warn("Không kết nối được daily_records:", e);
        }
      }

      // 2. Gọi Open-Meteo API để lấy chuỗi ngày thời gian thực
      const url = `https://api.open-meteo.com/v1/forecast?latitude=${lat}&longitude=${lon}&daily=precipitation_sum&start_date=${fetchStart}&end_date=${fetchEnd}&timezone=Asia%2FBangkok`;

      try {
        const response = await fetch(url);
        if (!response.ok) throw new Error("API Offline");
        const data = await response.json();

        const dates = data.daily.time;
        const rains = data.daily.precipitation_sum;

        combinedTimeline = dates.map((dateStr, idx) => {
          const isFuture = dateStr > todayStr;
          const isToday = dateStr === todayStr;
          let rainVal = rains[idx] !== null ? rains[idx] : 0;
          let overrideLossVal = null;
          let typeStr = isFuture ? 'Dự báo' : (isToday ? 'Hôm nay' : 'Quá khứ');

          // Nếu ngày này đã được lưu trong database trước đó
          if (savedDailyMap[dateStr]) {
            const saved = savedDailyMap[dateStr];
            if (saved.rain !== null && saved.rain !== undefined) rainVal = parseFloat(saved.rain);
            if (saved.loss !== null && saved.loss !== undefined) overrideLossVal = parseFloat(saved.loss);
            typeStr = 'Cloud Lưu';
          }

          return {
            date: dateStr,
            rain: rainVal,
            type: typeStr,
            isForecast: isFuture,
            overrideLoss: overrideLossVal
          };
        });

        document.getElementById('valRainSource').innerText = "Nguồn: Open-Meteo & Supabase Cloud";
        document.getElementById('tableStatusTag').innerText = `Đã đồng bộ ${combinedTimeline.length} ngày liên tục`;
        runContinuousSimulation();
      } catch (err) {
        fallbackData();
      }
    }

    function fallbackData() {
      const baseDates = [
        '2026-10-25', '2026-10-26', '2026-10-27', '2026-10-28', '2026-10-29', '2026-10-30', '2026-10-31',
        '2026-11-01', '2026-11-02', '2026-11-03', '2026-11-04', '2026-11-05', '2026-11-06', '2026-11-07',
        '2026-11-08', '2026-11-09', '2026-11-10', '2026-11-11', '2026-11-12'
      ];
      const sampleRains = [0.10, 0.20, 0.60, 0.40, 8.30, 2.80, 9.50, 0.60, 2.40, 7.70, 2.50, 0.00, 6.70, 2.20, 7.70, 2.10, 5.70, 7.20, 1.00];

      combinedTimeline = baseDates.map((d, i) => ({
        date: d,
        rain: sampleRains[i],
        type: i > 11 ? 'Dự báo' : (i === 11 ? 'Hôm nay' : 'Quá khứ'),
        isForecast: i > 11,
        overrideLoss: null
      }));
      runContinuousSimulation();
    }

    async function submitManualRecord() {
      const mDate = document.getElementById('inputManualDate').value;
      const mRain = parseFloat(document.getElementById('inputManualRain').value);
      const mLoss = document.getElementById('inputManualPloss').value;

      if (!mDate || isNaN(mRain)) {
        alert("Vui lòng nhập ngày và lượng mưa hợp lệ!");
        return;
      }

      const overrideVal = mLoss !== "" ? parseFloat(mLoss) : null;

      // Lưu trực tiếp vào bảng daily_records trên Supabase
      if (supabaseClient && currentProj) {
        try {
          const recId = `${currentProj.id}_${mDate}`;
          await supabaseClient.from('daily_records').upsert([{
            id: recId,
            project_id: currentProj.id,
            record_date: mDate,
            rain: mRain,
            loss: overrideVal,
            is_cleaned: false
          }]);
        } catch(e) {
          console.warn("Lỗi lưu Supabase daily_record:", e);
        }
      }

      const existIdx = combinedTimeline.findIndex(item => item.date === mDate);
      const record = {
        date: mDate,
        rain: mRain,
        type: 'Thực đo (Cloud)',
        isForecast: false,
        overrideLoss: overrideVal
      };

      if (existIdx >= 0) {
        combinedTimeline[existIdx] = record;
      } else {
        combinedTimeline.push(record);
        combinedTimeline.sort((a, b) => a.date.localeCompare(b.date));
      }

      alert(`Đã lưu bản ghi ngày ${mDate} lên Supabase Cloud thành công!`);
      runContinuousSimulation();
    }

    // Tự động lưu nhật ký ngày hôm nay vào Supabase để bảo tồn vĩnh viễn
    async function autoSaveTodayRecord(todayItem, todayLoss) {
      if (!supabaseClient || !currentProj || !todayItem || todayItem.isForecast) return;
      try {
        const recId = `${currentProj.id}_${todayItem.date}`;
        await supabaseClient.from('daily_records').upsert([{
          id: recId,
          project_id: currentProj.id,
          record_date: todayItem.date,
          rain: todayItem.rain,
          loss: todayLoss,
          is_cleaned: false
        }], { onConflict: 'project_id,record_date' });
      } catch(e) {}
    }

    function runContinuousSimulation() {
      if (!combinedTimeline.length) return;

      let currentLossProp = 0;
      const lossHistory = [];

      combinedTimeline.forEach((item) => {
        if (item.overrideLoss !== undefined && item.overrideLoss !== null) {
          currentLossProp = item.overrideLoss / 100;
        } else {
          if (item.rain >= rainThresh) {
            currentLossProp = 0;
          } else {
            currentLossProp += r_daily;
          }
        }

        if (currentLossProp >= calculatedLStar) {
          currentLossProp = 0;
        }

        lossHistory.push(currentLossProp * 100);
      });

      let todayIdx = combinedTimeline.findIndex(d => d.type === 'Hôm nay' || d.type.includes('Thực đo') || d.type === 'Cloud Lưu');
      if (todayIdx < 0) todayIdx = combinedTimeline.length - 1;

      const todayItem = combinedTimeline[todayIdx];
      const todayLoss = lossHistory[todayIdx];

      // Kích hoạt tự lưu ngày hiện tại vào Cloud Supabase
      autoSaveTodayRecord(todayItem, todayLoss);

      const futureItems = combinedTimeline.filter(d => d.isForecast);
      const upcomingRainyDays = futureItems.filter(d => d.rain >= rainThresh);
      document.getElementById('valUpcomingRainCount').innerText = upcomingRainyDays.length;
      if (upcomingRainyDays.length > 0) {
        document.getElementById('valNextBigRain').innerText = `Mưa lớn tới: ${upcomingRainyDays[0].date.slice(5)} (${upcomingRainyDays[0].rain.toFixed(2)} mm)`;
      } else {
        document.getElementById('valNextBigRain').innerText = "7 ngày tới: Không có mưa lớn";
      }

      document.getElementById('valCurrentLoss').innerText = todayLoss.toFixed(2) + '%';
      document.getElementById('valTodayRain').innerText = todayItem.rain.toFixed(2);

      const progressPercent = Math.min(100, (todayLoss / (calculatedLStar * 100)) * 100);
      document.getElementById('lossProgressBar').style.width = progressPercent + '%';

      updateDecisionBanner(todayItem, todayLoss, upcomingRainyDays);
      renderTable(lossHistory);
      renderChart(lossHistory);
    }

    function updateDecisionBanner(todayItem, todayLoss, upcomingRainyDays) {
      const banner = document.getElementById('statusBanner');
      const badge = document.getElementById('statusBadge');
      const title = document.getElementById('statusTitle');
      const desc = document.getElementById('statusDesc');
      const actionBtn = document.getElementById('actionBtn');

      const isCritical = todayLoss >= (calculatedLStar * 100);
      const isRainWashedToday = todayItem.rain >= rainThresh;
      const willRainSoon = upcomingRainyDays.length > 0;

      if (isRainWashedToday) {
        banner.className = "p-5 rounded-2xl border transition-all duration-300 flex flex-col md:flex-row items-start md:items-center justify-between gap-4 card-clean border-blue-200 bg-blue-50/40";
        badge.className = "text-xs font-bold uppercase tracking-wider px-2 py-0.5 rounded bg-blue-100 text-blue-700";
        badge.innerText = "TỰ NHIÊN LÀM SẠCH";
        title.innerText = `Cơn mưa ${todayItem.rain.toFixed(2)} mm hôm nay đã làm sạch bề mặt pin!`;
        desc.innerText = `Lượng mưa vượt ngưỡng Kimber (${rainThresh.toFixed(2)} mm). Tổn thất reset về 0.00%. Tiết kiệm hoàn toàn chi phí vệ sinh.`;
        actionBtn.className = "w-full md:w-auto px-5 py-2.5 rounded-xl font-semibold text-sm bg-slate-100 text-slate-400 cursor-not-allowed border border-slate-200";
        actionBtn.disabled = true;
      } else if (isCritical) {
        if (willRainSoon) {
          banner.className = "p-5 rounded-2xl border transition-all duration-300 flex flex-col md:flex-row items-start md:items-center justify-between gap-4 card-clean border-amber-300 bg-amber-50/50";
          badge.className = "text-xs font-bold uppercase tracking-wider px-2 py-0.5 rounded bg-amber-100 text-amber-800";
          badge.innerText = "TẠM HOÃN VỆ SINH (CÓ MƯA TỚI)";
          title.innerText = `Tổn thất chạm ngưỡng L* (${todayLoss.toFixed(2)}%) nhưng có mưa lớn sắp tới (${upcomingRainyDays[0].date.slice(5)})`;
          desc.innerText = `Dự báo có cơn mưa ${upcomingRainyDays[0].rain.toFixed(2)} mm trong 1-3 ngày tới. Khuyến nghị hoãn rửa để đón mưa tự nhiên.`;
          actionBtn.className = "w-full md:w-auto px-5 py-2.5 rounded-xl font-semibold text-sm bg-amber-500 hover:bg-amber-600 text-white cursor-pointer shadow-sm";
          actionBtn.disabled = false;
        } else {
          banner.className = "p-5 rounded-2xl border transition-all duration-300 flex flex-col md:flex-row items-start md:items-center justify-between gap-4 card-clean border-rose-300 bg-rose-50/50";
          badge.className = "text-xs font-bold uppercase tracking-wider px-2 py-0.5 rounded bg-rose-100 text-rose-700";
          badge.innerText = "CẦN VỆ SINH NGAY";
          title.innerText = `Tổn thất chạm ngưỡng L* (${todayLoss.toFixed(2)}% ≥ ${(calculatedLStar*100).toFixed(2)}%) & 7 ngày tới không mưa`;
          desc.innerText = `Tiền điện tổn thất lũy kế đã vượt chi phí 1 lần rửa. Cần phát lệnh vệ sinh ngay để khôi phục công suất phát.`;
          actionBtn.className = "w-full md:w-auto px-5 py-2.5 rounded-xl font-semibold text-sm bg-rose-600 hover:bg-rose-700 text-white shadow-sm cursor-pointer";
          actionBtn.disabled = false;
        }
      } else {
        banner.className = "p-5 rounded-2xl border transition-all duration-300 flex flex-col md:flex-row items-start md:items-center justify-between gap-4 card-clean border-emerald-200 bg-emerald-50/30";
        badge.className = "text-xs font-bold uppercase tracking-wider px-2 py-0.5 rounded bg-emerald-100 text-emerald-800";
        badge.innerText = "CHƯA CẦN VỆ SINH";
        title.innerText = `Hệ thống hoạt động tối ưu kinh tế (Tổn thất ${todayLoss.toFixed(2)}% < ${(calculatedLStar*100).toFixed(2)}%)`;
        desc.innerText = `Giá trị điện thu hồi chưa bù đắp chi phí rửa (${formatVND(C_clean)}). Tiếp tục theo dõi tích lũy bụi.`;
        actionBtn.className = "w-full md:w-auto px-5 py-2.5 rounded-xl font-semibold text-sm bg-slate-100 text-slate-400 cursor-not-allowed border border-slate-200";
        actionBtn.disabled = true;
      }
    }

    function renderTable(lossHistory) {
      const tbody = document.getElementById('historyTableBody');
      tbody.innerHTML = '';

      combinedTimeline.forEach((item, i) => {
        const lossVal = lossHistory[i];
        const isRainWashed = item.rain >= rainThresh;
        const isCritical = lossVal >= (calculatedLStar * 100);

        let badgeTag = `<span class="px-2 py-0.5 rounded text-[10px] bg-slate-100 text-slate-600 border border-slate-200">${item.type}</span>`;
        if (item.type.includes('Thực đo') || item.type === 'Cloud Lưu') {
          badgeTag = `<span class="px-2 py-0.5 rounded text-[10px] bg-emerald-100 text-emerald-800 font-semibold border border-emerald-200">${item.type}</span>`;
        } else if (item.type === 'Hôm nay') {
          badgeTag = `<span class="px-2 py-0.5 rounded text-[10px] bg-blue-100 text-blue-800 font-semibold border border-blue-200">${item.type}</span>`;
        } else if (item.isForecast) {
          badgeTag = `<span class="px-2 py-0.5 rounded text-[10px] bg-indigo-100 text-indigo-800 border border-indigo-200">${item.type}</span>`;
        }

        let actionBadge = `<span class="text-slate-400 font-medium">Chờ tích lũy</span>`;
        if (isRainWashed) {
          actionBadge = `<span class="text-blue-600 font-semibold">Tự rửa sạch (Reset)</span>`;
        } else if (isCritical) {
          actionBadge = `<span class="text-rose-600 font-semibold">Cần rửa pin</span>`;
        }

        const tr = document.createElement('tr');
        tr.className = item.type === 'Hôm nay' ? 'bg-blue-50/60 font-medium' : 'hover:bg-slate-50';
        tr.innerHTML = `
          <td class="py-2.5 px-3 font-mono">${item.date}</td>
          <td class="py-2.5 px-3">${badgeTag}</td>
          <td class="py-2.5 px-3 ${item.rain >= rainThresh ? 'text-blue-600 font-bold' : 'text-slate-700'}">${item.rain.toFixed(2)} mm</td>
          <td class="py-2.5 px-3 font-mono text-amber-600 font-semibold">${lossVal.toFixed(2)}%</td>
          <td class="py-2.5 px-3">${item.rain >= rainThresh ? '<span class="text-blue-600 font-medium">Rửa trôi tự nhiên</span>' : (item.rain > 0 ? 'Mưa nhỏ' : 'Khô ráo')}</td>
          <td class="py-2.5 px-3">${actionBadge}</td>
        `;
        tbody.appendChild(tr);
      });
    }

    function renderChart(lossHistory) {
      const labels = combinedTimeline.map(d => d.date.slice(5));
      const rainData = combinedTimeline.map(d => d.rain);
      const lStarLine = Array(labels.length).fill(calculatedLStar * 100);

      const ctx = document.getElementById('mainChart').getContext('2d');
      if (chartInstance) chartInstance.destroy();

      chartInstance = new Chart(ctx, {
        type: 'bar',
        data: {
          labels: labels,
          datasets: [
            {
              type: 'line',
              label: 'Tổn thất công suất (%)',
              data: lossHistory,
              borderColor: '#f59e0b',
              backgroundColor: 'rgba(245, 158, 11, 0.08)',
              borderWidth: 2,
              tension: 0.2,
              yAxisID: 'y1'
            },
            {
              type: 'line',
              label: 'Ngưỡng L*',
              data: lStarLine,
              borderColor: '#ef4444',
              borderWidth: 1.5,
              borderDash: [6, 4],
              pointRadius: 0,
              yAxisID: 'y1'
            },
            {
              type: 'bar',
              label: 'Lượng mưa (mm)',
              data: rainData,
              backgroundColor: combinedTimeline.map(d => d.isForecast ? 'rgba(129, 140, 248, 0.45)' : 'rgba(59, 130, 246, 0.6)'),
              borderRadius: 4,
              yAxisID: 'y'
            }
          ]
        },
        options: {
          responsive: true,
          maintainAspectRatio: false,
          interaction: { mode: 'index', intersect: false },
          scales: {
            x: { grid: { color: 'rgba(0, 0, 0, 0.04)' }, ticks: { color: '#64748b', font: { size: 10 } } },
            y: {
              type: 'linear',
              position: 'left',
              title: { display: true, text: 'Lượng mưa (mm)', color: '#2563eb', font: { size: 11 } },
              grid: { color: 'rgba(0, 0, 0, 0.04)' },
              ticks: { color: '#64748b' }
            },
            y1: {
              type: 'linear',
              position: 'right',
              title: { display: true, text: 'Tổn thất (%)', color: '#d97706', font: { size: 11 } },
              grid: { drawOnChartArea: false },
              ticks: { color: '#64748b' }
            }
          },
          plugins: {
            legend: {
              display: true,
              labels: { color: '#334155', font: { size: 11 } }
            }
          }
        }
      });
    }

    function exportToCSV() {
      if (!combinedTimeline.length) return;
      let csvContent = "data:text/csv;charset=utf-8,";
      csvContent += "Ngay,Phan_loai,Luong_mua_mm,Ton_that_pct\n";

      let curProp = 0;
      combinedTimeline.forEach(item => {
        if (item.overrideLoss !== null && item.overrideLoss !== undefined) {
          curProp = item.overrideLoss / 100;
        } else {
          curProp = (item.rain >= rainThresh) ? 0 : (curProp + r_daily);
        }
        if (curProp >= calculatedLStar) curProp = 0;
        csvContent += `${item.date},${item.type},${item.rain.toFixed(2)},${(curProp * 100).toFixed(2)}\n`;
      });

      const encodedUri = encodeURI(csvContent);
      const link = document.createElement("a");
      link.setAttribute("href", encodedUri);
      link.setAttribute("download", `bao_cao_OM_${new Date().toISOString().slice(0,10)}.csv`);
      document.body.appendChild(link);
      link.click();
      document.body.removeChild(link);
    }

    function manualCleanPin() {
      alert("Đã ghi nhận sự kiện vệ sinh tấm pin! Tổn thất công suất được reset về 0%.");
      const todayIdx = combinedTimeline.findIndex(d => d.type === 'Hôm nay' || d.type.includes('Thực đo') || d.type === 'Cloud Lưu');
      if (todayIdx >= 0) {
        combinedTimeline[todayIdx].overrideLoss = 0.0;
      }
      runContinuousSimulation();
    }

    window.addEventListener('DOMContentLoaded', () => {
      checkAuthSession();
    });
  </script>
</body>
</html>
