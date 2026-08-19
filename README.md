<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Sistem Absensi Siswa QR Code</title>
    <!-- Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- FontAwesome Icons -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <!-- Chart.js -->
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <!-- QRCode.js Generator -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/qrcodejs/1.0.0/qrcode.min.js"></script>
    <!-- HTML5-QRCode Scanner -->
    <script src="https://unpkg.com/html5-qrcode"></script>

    <script>
        tailwind.config = {
            theme: {
                extend: {
                    colors: {
                        brand: {
                            50: '#f0fdf4',
                            100: '#dcfce7',
                            500: '#22c55e',
                            600: '#16a34a',
                            700: '#15803d',
                            800: '#166534',
                        }
                    }
                }
            }
        }
    </script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700&display=swap');
        body { font-family: 'Inter', sans-serif; }
        .no-scrollbar::-webkit-scrollbar { display: none; }
        .no-scrollbar { -ms-overflow-style: none; scrollbar-width: none; }
    </style>
</head>
<body class="bg-slate-100 text-slate-800 antialiased min-h-screen flex flex-col md:flex-row">

    <!-- Sidebar Navigation -->
    <aside class="w-full md:w-64 bg-slate-900 text-white flex-shrink-0 flex flex-col justify-between z-20 shadow-lg">
        <div>
            <!-- Logo Header -->
            <div class="p-5 flex items-center gap-3 border-b border-slate-800">
                <img id="appLogo" src="https://cdn-icons-png.flaticon.com/512/2991/2991108.png" alt="Logo" class="w-10 h-10 object-contain bg-white rounded-lg p-1">
                <div>
                    <h1 id="appSchoolName" class="font-bold text-sm leading-tight text-emerald-400 line-clamp-1">SD Negeri 01</h1>
                    <p class="text-xs text-slate-400">Presensi & QR Code</p>
                </div>
            </div>

            <!-- Menu Navigation -->
            <nav class="p-4 space-y-1">
                <button onclick="switchTab('dashboard')" id="nav-dashboard" class="nav-btn w-full flex items-center gap-3 px-4 py-3 rounded-xl text-sm font-medium transition duration-200 bg-emerald-600 text-white">
                    <i class="fa-solid fa-chart-pie w-5"></i> Dashboard
                </button>
                <button onclick="switchTab('kelas')" id="nav-kelas" class="nav-btn w-full flex items-center gap-3 px-4 py-3 rounded-xl text-sm font-medium transition duration-200 text-slate-300 hover:bg-slate-800 hover:text-white">
                    <i class="fa-solid fa-school w-5"></i> Daftar Kelas
                </button>
                <button onclick="switchTab('siswa')" id="nav-siswa" class="nav-btn w-full flex items-center gap-3 px-4 py-3 rounded-xl text-sm font-medium transition duration-200 text-slate-300 hover:bg-slate-800 hover:text-white">
                    <i class="fa-solid fa-user-graduate w-5"></i> Daftar Siswa
                </button>
                <button onclick="switchTab('absen')" id="nav-absen" class="nav-btn w-full flex items-center gap-3 px-4 py-3 rounded-xl text-sm font-medium transition duration-200 text-slate-300 hover:bg-slate-800 hover:text-white">
                    <i class="fa-solid fa-clipboard-user w-5"></i> Absen / Scan QR
                </button>
                <button onclick="switchTab('rekap')" id="nav-rekap" class="nav-btn w-full flex items-center gap-3 px-4 py-3 rounded-xl text-sm font-medium transition duration-200 text-slate-300 hover:bg-slate-800 hover:text-white">
                    <i class="fa-solid fa-file-invoice w-5"></i> Rekap Absen
                </button>
                <button onclick="switchTab('pengaturan')" id="nav-pengaturan" class="nav-btn w-full flex items-center gap-3 px-4 py-3 rounded-xl text-sm font-medium transition duration-200 text-slate-300 hover:bg-slate-800 hover:text-white">
                    <i class="fa-solid fa-gear w-5"></i> Pengaturan
                </button>
            </nav>
        </div>

        <!-- System Status Bar -->
        <div class="p-4 border-t border-slate-800 text-xs text-slate-400 flex items-center justify-between">
            <span class="flex items-center gap-2">
                <span id="statusDot" class="w-2 h-2 rounded-full bg-emerald-500 animate-pulse"></span>
                <span id="statusText">Terhubung</span>
            </span>
            <button onclick="loadInitialData()" class="hover:text-white" title="Refresh Data"><i class="fa-solid fa-rotate-right"></i></button>
        </div>
    </aside>

    <!-- Main Content Area -->
    <main class="flex-1 flex flex-col h-screen overflow-y-auto">
        
        <!-- Top App Bar -->
        <header class="bg-white border-b border-slate-200 px-6 py-4 flex items-center justify-between sticky top-0 z-10">
            <div>
                <h2 id="pageTitle" class="text-xl font-bold text-slate-800">Dashboard Statistik</h2>
                <p id="pageSubTitle" class="text-xs text-slate-500">Ringkasan statistik kehadiran siswa hari ini</p>
            </div>
            <div class="flex items-center gap-3">
                <div class="text-right hidden sm:block">
                    <p id="topGuruName" class="text-sm font-semibold text-slate-800">Guru / Wali Kelas</p>
                    <p id="topGuruNip" class="text-xs text-slate-500">NIP. -</p>
                </div>
                <div class="w-9 h-9 rounded-full bg-emerald-100 text-emerald-700 font-bold flex items-center justify-center">
                    <i class="fa-solid fa-user"></i>
                </div>
            </div>
        </header>

        <!-- Content Views Wrapper -->
        <div class="p-6 flex-1 space-y-6">

            <!-- 1. DASHBOARD VIEW -->
            <section id="view-dashboard" class="tab-view space-y-6">
                <!-- Summary Cards -->
                <div class="grid grid-cols-2 md:grid-cols-5 gap-4">
                    <div class="bg-white p-4 rounded-2xl border border-slate-200 shadow-sm flex items-center gap-3">
                        <div class="w-12 h-12 rounded-xl bg-blue-100 text-blue-600 flex items-center justify-center text-xl font-bold">
                            <i class="fa-solid fa-users"></i>
                        </div>
                        <div>
                            <p class="text-xs text-slate-500 font-medium">Total Siswa</p>
                            <h3 id="dashTotalSiswa" class="text-2xl font-bold text-slate-800">0</h3>
                        </div>
                    </div>

                    <div class="bg-white p-4 rounded-2xl border border-slate-200 shadow-sm flex items-center gap-3">
                        <div class="w-12 h-12 rounded-xl bg-emerald-100 text-emerald-600 flex items-center justify-center text-xl font-bold">
                            <i class="fa-solid fa-user-check"></i>
                        </div>
                        <div>
                            <p class="text-xs text-slate-500 font-medium">Hadir (H)</p>
                            <h3 id="dashHadir" class="text-2xl font-bold text-emerald-600">0</h3>
                        </div>
                    </div>

                    <div class="bg-white p-4 rounded-2xl border border-slate-200 shadow-sm flex items-center gap-3">
                        <div class="w-12 h-12 rounded-xl bg-amber-100 text-amber-600 flex items-center justify-center text-xl font-bold">
                            <i class="fa-solid fa-envelope-open-text"></i>
                        </div>
                        <div>
                            <p class="text-xs text-slate-500 font-medium">Izin (I)</p>
                            <h3 id="dashIzin" class="text-2xl font-bold text-amber-600">0</h3>
                        </div>
                    </div>

                    <div class="bg-white p-4 rounded-2xl border border-slate-200 shadow-sm flex items-center gap-3">
                        <div class="w-12 h-12 rounded-xl bg-indigo-100 text-indigo-600 flex items-center justify-center text-xl font-bold">
                            <i class="fa-solid fa-notes-medical"></i>
                        </div>
                        <div>
                            <p class="text-xs text-slate-500 font-medium">Sakit (S)</p>
                            <h3 id="dashSakit" class="text-2xl font-bold text-indigo-600">0</h3>
                        </div>
                    </div>

                    <div class="bg-white p-4 rounded-2xl border border-slate-200 shadow-sm flex items-center gap-3 col-span-2 md:col-span-1">
                        <div class="w-12 h-12 rounded-xl bg-rose-100 text-rose-600 flex items-center justify-center text-xl font-bold">
                            <i class="fa-solid fa-user-xmark"></i>
                        </div>
                        <div>
                            <p class="text-xs text-slate-500 font-medium">Alpa (A)</p>
                            <h3 id="dashAlpa" class="text-2xl font-bold text-rose-600">0</h3>
                        </div>
                    </div>
                </div>

                <!-- Chart & Activity Grid -->
                <div class="grid grid-cols-1 lg:grid-cols-3 gap-6">
                    <div class="lg:col-span-2 bg-white p-6 rounded-2xl border border-slate-200 shadow-sm">
                        <h4 class="text-sm font-bold text-slate-800 mb-4 flex items-center gap-2">
                            <i class="fa-solid fa-chart-column text-emerald-600"></i> Persentase Kehadiran Hari Ini
                        </h4>
                        <div class="h-64">
                            <canvas id="attendanceChart"></canvas>
                        </div>
                    </div>

                    <div class="bg-white p-6 rounded-2xl border border-slate-200 shadow-sm flex flex-col justify-between">
                        <div>
                            <h4 class="text-sm font-bold text-slate-800 mb-4 flex items-center gap-2">
                                <i class="fa-solid fa-clock-rotate-left text-blue-600"></i> Absensi Terakhir
                            </h4>
                            <div id="recentAttendanceList" class="space-y-3 text-sm overflow-y-auto max-h-60 no-scrollbar">
                                <p class="text-xs text-slate-400 italic">Belum ada data presensi hari ini.</p>
                            </div>
                        </div>
                        <button onclick="switchTab('absen')" class="w-full mt-4 py-2 bg-emerald-50 text-emerald-700 hover:bg-emerald-100 font-medium text-xs rounded-xl transition">
                            <i class="fa-solid fa-qrcode mr-1"></i> Buka Kamera Scanner Absen
                        </button>
                    </div>
                </div>
            </section>

            <!-- 2. DAFTAR KELAS VIEW -->
            <section id="view-kelas" class="tab-view hidden space-y-6">
                <div class="grid grid-cols-1 md:grid-cols-3 gap-6">
                    <!-- Form Input Kelas -->
                    <div class="bg-white p-6 rounded-2xl border border-slate-200 shadow-sm h-fit">
                        <h3 class="text-base font-bold text-slate-800 mb-4 flex items-center gap-2">
                            <i class="fa-solid fa-plus-circle text-emerald-600"></i> Tambah Kelas
                        </h3>
                        <form id="formKelas" onsubmit="handleSaveClass(event)" class="space-y-4">
                            <div>
                                <label class="block text-xs font-semibold text-slate-600 mb-1">Nama Kelas</label>
                                <input type="text" id="kelasNama" placeholder="Contoh: X IPA 1" required class="w-full px-3 py-2 text-sm border border-slate-300 rounded-xl focus:ring-2 focus:ring-emerald-500 focus:outline-none">
                            </div>
                            <div>
                                <label class="block text-xs font-semibold text-slate-600 mb-1">Wali Kelas</label>
                                <input type="text" id="kelasWali" placeholder="Nama Guru Wali Kelas" required class="w-full px-3 py-2 text-sm border border-slate-300 rounded-xl focus:ring-2 focus:ring-emerald-500 focus:outline-none">
                            </div>
                            <button type="submit" class="w-full bg-emerald-600 hover:bg-emerald-700 text-white font-medium py-2.5 rounded-xl text-sm transition flex items-center justify-center gap-2 shadow-sm">
                                <i class="fa-solid fa-save"></i> Simpan Kelas
                            </button>
                        </form>
                    </div>

                    <!-- Table Kelas -->
                    <div class="md:col-span-2 bg-white rounded-2xl border border-slate-200 shadow-sm overflow-hidden">
                        <div class="p-4 border-b border-slate-100 flex justify-between items-center bg-slate-50">
                            <h3 class="font-bold text-sm text-slate-800">Daftar Kelas Terdaftar</h3>
                            <span id="countKelas" class="text-xs bg-emerald-100 text-emerald-700 font-semibold px-2.5 py-1 rounded-full">0 Kelas</span>
                        </div>
                        <div class="overflow-x-auto">
                            <table class="w-full text-left border-collapse text-sm">
                                <thead>
                                    <tr class="bg-slate-100 text-slate-600 text-xs uppercase font-semibold">
                                        <th class="p-3">ID Kelas</th>
                                        <th class="p-3">Nama Kelas</th>
                                        <th class="p-3">Wali Kelas</th>
                                        <th class="p-3 text-center">Aksi</th>
                                    </tr>
                                </thead>
                                <tbody id="tableKelasBody" class="divide-y divide-slate-100 text-slate-700">
                                    <!-- Rows -->
                                </tbody>
                            </table>
                        </div>
                    </div>
                </div>
            </section>

            <!-- 3. DAFTAR SISWA VIEW -->
            <section id="view-siswa" class="tab-view hidden space-y-6">
                <!-- Top Toolbar -->
                <div class="bg-white p-4 rounded-2xl border border-slate-200 shadow-sm flex flex-wrap items-center justify-between gap-4">
                    <div class="flex items-center gap-3 w-full md:w-auto">
                        <select id="filterKelasSiswa" onchange="renderSiswaTable()" class="px-3 py-2 border border-slate-300 rounded-xl text-sm focus:outline-none focus:ring-2 focus:ring-emerald-500">
                            <option value="">-- Semua Kelas --</option>
                        </select>
                        <div class="relative w-full md:w-64">
                            <i class="fa-solid fa-search absolute left-3 top-3 text-slate-400 text-xs"></i>
                            <input type="text" id="searchSiswa" onkeyup="renderSiswaTable()" placeholder="Cari NIS / Nama Siswa..." class="w-full pl-8 pr-3 py-2 text-sm border border-slate-300 rounded-xl focus:outline-none focus:ring-2 focus:ring-emerald-500">
                        </div>
                    </div>
                    <button onclick="openModalSiswa()" class="bg-emerald-600 hover:bg-emerald-700 text-white font-medium px-4 py-2 rounded-xl text-sm transition flex items-center gap-2 shadow-sm">
                        <i class="fa-solid fa-user-plus"></i> Tambah Siswa Baru
                    </button>
                </div>

                <!-- Table Siswa -->
                <div class="bg-white rounded-2xl border border-slate-200 shadow-sm overflow-hidden">
                    <div class="overflow-x-auto">
                        <table class="w-full text-left border-collapse text-sm">
                            <thead>
                                <tr class="bg-slate-100 text-slate-600 text-xs uppercase font-semibold">
                                    <th class="p-3">NIS / NISN</th>
                                    <th class="p-3">Nama Lengkap</th>
                                    <th class="p-3">Kelas</th>
                                    <th class="p-3">JK</th>
                                    <th class="p-3 text-center">Kartu QR Code</th>
                                    <th class="p-3 text-center">Aksi</th>
                                </tr>
                            </thead>
                            <tbody id="tableSiswaBody" class="divide-y divide-slate-100 text-slate-700">
                                <!-- Dynamic rows -->
                            </tbody>
                        </table>
                    </div>
                </div>
            </section>

            <!-- 4. ABSEN VIEW (MANUAL & SCANNER) -->
            <section id="view-absen" class="tab-view hidden space-y-6">
                <!-- Mode Switcher & Filter -->
                <div class="bg-white p-4 rounded-2xl border border-slate-200 shadow-sm flex flex-wrap items-center justify-between gap-4">
                    <div class="flex items-center gap-3">
                        <label class="text-xs font-semibold text-slate-600">Pilih Kelas:</label>
                        <select id="absenFilterKelas" onchange="loadAbsenSiswaTable()" class="px-3 py-2 border border-slate-300 rounded-xl text-sm focus:outline-none focus:ring-2 focus:ring-emerald-500">
                            <!-- Dynamic Class Options -->
                        </select>
                        <input type="date" id="absenTanggal" onchange="loadAbsenSiswaTable()" class="px-3 py-2 border border-slate-300 rounded-xl text-sm focus:outline-none focus:ring-2 focus:ring-emerald-500">
                    </div>

                    <div class="flex items-center bg-slate-100 p-1 rounded-xl">
                        <button id="btnModeManual" onclick="setAbsenMode('manual')" class="px-4 py-1.5 rounded-lg text-xs font-semibold text-emerald-700 bg-white shadow-sm transition">
                            <i class="fa-solid fa-list-check mr-1"></i> Input Manual
                        </button>
                        <button id="btnModeScan" onclick="setAbsenMode('scan')" class="px-4 py-1.5 rounded-lg text-xs font-semibold text-slate-600 hover:text-slate-800 transition">
                            <i class="fa-solid fa-qrcode mr-1"></i> Scan Barcode / QR
                        </button>
                    </div>
                </div>

                <!-- Scanner Container -->
                <div id="containerScanner" class="hidden bg-slate-900 rounded-2xl p-6 text-white text-center max-w-lg mx-auto shadow-xl relative">
                    <h3 class="text-base font-bold mb-1 flex items-center justify-center gap-2 text-emerald-400">
                        <i class="fa-solid fa-camera"></i> Pemindai QR Code Siswa
                    </h3>
                    <p class="text-xs text-slate-400 mb-4">Arahkan QR Code Kartu Siswa ke Kamera</p>
                    
                    <div id="reader" class="overflow-hidden rounded-xl bg-black max-w-sm mx-auto border-2 border-emerald-500"></div>

                    <!-- Scan Feedback Log -->
                    <div id="scanFeedback" class="mt-4 p-3 rounded-xl bg-slate-800 text-xs text-emerald-300 border border-slate-700 min-h-[48px] flex items-center justify-center">
                        Siap melakukan pemindaian...
                    </div>
                </div>

                <!-- Manual Table Container -->
                <div id="containerManual" class="bg-white rounded-2xl border border-slate-200 shadow-sm overflow-hidden">
                    <div class="p-4 border-b border-slate-100 flex justify-between items-center bg-slate-50">
                        <span class="text-xs text-slate-500">Pilih status kehadiran untuk tiap siswa: <b class="text-emerald-700">H = Hadir</b>, <b class="text-amber-700">I = Izin</b>, <b class="text-indigo-700">S = Sakit</b>, <b class="text-rose-700">A = Alpa</b></span>
                        <button onclick="submitAttendanceBatch()" class="bg-emerald-600 hover:bg-emerald-700 text-white text-xs font-bold px-4 py-2 rounded-xl transition shadow-sm">
                            <i class="fa-solid fa-floppy-disk mr-1"></i> Simpan Presensi Kelas
                        </button>
                    </div>
                    <div class="overflow-x-auto">
                        <table class="w-full text-left border-collapse text-sm">
                            <thead>
                                <tr class="bg-slate-100 text-slate-600 text-xs uppercase font-semibold">
                                    <th class="p-3">No</th>
                                    <th class="p-3">NIS</th>
                                    <th class="p-3">Nama Siswa</th>
                                    <th class="p-3 text-center">Status Kehadiran</th>
                                    <th class="p-3">Keterangan</th>
                                </tr>
                            </thead>
                            <tbody id="tableAbsenBody" class="divide-y divide-slate-100 text-slate-700">
                                <!-- Dynamic Rows -->
                            </tbody>
                        </table>
                    </div>
                </div>
            </section>

            <!-- 5. REKAP ABSEN VIEW -->
            <section id="view-rekap" class="tab-view hidden space-y-6">
                <!-- Filter Controls -->
                <div class="bg-white p-6 rounded-2xl border border-slate-200 shadow-sm space-y-4">
                    <div class="flex flex-wrap items-center justify-between gap-4">
                        <div class="flex items-center gap-4">
                            <div>
                                <label class="block text-xs font-semibold text-slate-600 mb-1">Tipe Rekap</label>
                                <select id="rekapType" onchange="toggleRekapType()" class="px-3 py-2 border border-slate-300 rounded-xl text-sm focus:outline-none focus:ring-2 focus:ring-emerald-500">
                                    <option value="harian">Rekap Harian</option>
                                    <option value="bulanan">Rekap Bulanan</option>
                                </select>
                            </div>

                            <div>
                                <label class="block text-xs font-semibold text-slate-600 mb-1">Kelas</label>
                                <select id="rekapKelas" class="px-3 py-2 border border-slate-300 rounded-xl text-sm focus:outline-none focus:ring-2 focus:ring-emerald-500">
                                    <!-- Options -->
                                </select>
                            </div>

                            <div id="wrapRekapTgl">
                                <label class="block text-xs font-semibold text-slate-600 mb-1">Tanggal</label>
                                <input type="date" id="rekapTanggal" class="px-3 py-2 border border-slate-300 rounded-xl text-sm focus:outline-none focus:ring-2 focus:ring-emerald-500">
                            </div>

                            <div id="wrapRekapBulan" class="hidden flex gap-2">
                                <div>
                                    <label class="block text-xs font-semibold text-slate-600 mb-1">Bulan</label>
                                    <select id="rekapBulan" class="px-3 py-2 border border-slate-300 rounded-xl text-sm focus:outline-none focus:ring-2 focus:ring-emerald-500">
                                        <option value="01">Januari</option>
                                        <option value="02">Februari</option>
                                        <option value="03">Maret</option>
                                        <option value="04">April</option>
                                        <option value="05">Mei</option>
                                        <option value="06">Juni</option>
                                        <option value="07">Juli</option>
                                        <option value="08">Agustus</option>
                                        <option value="09">September</option>
                                        <option value="10">Oktober</option>
                                        <option value="11">November</option>
                                        <option value="12">Desember</option>
                                    </select>
                                </div>
                                <div>
                                    <label class="block text-xs font-semibold text-slate-600 mb-1">Tahun</label>
                                    <input type="number" id="rekapTahun" value="2026" class="w-24 px-3 py-2 border border-slate-300 rounded-xl text-sm focus:outline-none focus:ring-2 focus:ring-emerald-500">
                                </div>
                            </div>
                        </div>

                        <div class="flex items-center gap-2">
                            <button onclick="generateRekapView()" class="bg-slate-800 hover:bg-slate-900 text-white font-medium px-4 py-2 rounded-xl text-sm transition">
                                <i class="fa-solid fa-filter mr-1"></i> Tampilkan
                            </button>
                            <button onclick="printRekapNewTab()" class="bg-emerald-600 hover:bg-emerald-700 text-white font-medium px-4 py-2 rounded-xl text-sm transition shadow-sm">
                                <i class="fa-solid fa-print mr-1"></i> Cetak Rekap (Tab Baru)
                            </button>
                        </div>
                    </div>
                </div>

                <!-- Preview Area -->
                <div class="bg-white rounded-2xl border border-slate-200 shadow-sm p-6">
                    <div id="rekapPreviewHeader" class="text-center mb-6 pb-4 border-b border-slate-200">
                        <h3 id="rekapTitleText" class="font-bold text-lg text-slate-800 uppercase">REKAPITULASI PRESENSI SISWA</h3>
                        <p id="rekapSubText" class="text-xs text-slate-500">Periode Data Presensi</p>
                    </div>

                    <div class="overflow-x-auto">
                        <table id="tableRekap" class="w-full text-left border-collapse text-xs">
                            <!-- Dynamic Rekap Grid -->
                        </table>
                    </div>
                </div>
            </section>

            <!-- 6. PENGATURAN VIEW -->
            <section id="view-pengaturan" class="tab-view hidden space-y-6">
                <div class="bg-white p-6 rounded-2xl border border-slate-200 shadow-sm max-w-3xl mx-auto">
                    <h3 class="text-base font-bold text-slate-800 mb-4 pb-3 border-b border-slate-100 flex items-center gap-2">
                        <i class="fa-solid fa-sliders text-emerald-600"></i> Pengaturan Identitas Sekolah & Guru
                    </h3>

                    <form onsubmit="handleSaveSettings(event)" class="space-y-4">
                        <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
                            <div>
                                <label class="block text-xs font-semibold text-slate-600 mb-1">Nama Sekolah</label>
                                <input type="text" id="cfgNamaSekolah" required class="w-full px-3 py-2 text-sm border border-slate-300 rounded-xl focus:ring-2 focus:ring-emerald-500 focus:outline-none">
                            </div>
                            <div>
                                <label class="block text-xs font-semibold text-slate-600 mb-1">URL Logo Sekolah (Gambar)</label>
                                <input type="url" id="cfgLogoSekolah" required class="w-full px-3 py-2 text-sm border border-slate-300 rounded-xl focus:ring-2 focus:ring-emerald-500 focus:outline-none">
                            </div>
                        </div>

                        <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
                            <div>
                                <label class="block text-xs font-semibold text-slate-600 mb-1">Nama Guru / Wali Kelas</label>
                                <input type="text" id="cfgNamaGuru" required class="w-full px-3 py-2 text-sm border border-slate-300 rounded-xl focus:ring-2 focus:ring-emerald-500 focus:outline-none">
                            </div>
                            <div>
                                <label class="block text-xs font-semibold text-slate-600 mb-1">NIP Guru</label>
                                <input type="text" id="cfgNipGuru" required class="w-full px-3 py-2 text-sm border border-slate-300 rounded-xl focus:ring-2 focus:ring-emerald-500 focus:outline-none">
                            </div>
                        </div>

                        <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
                            <div>
                                <label class="block text-xs font-semibold text-slate-600 mb-1">Nama Kepala Sekolah</label>
                                <input type="text" id="cfgNamaKepsek" required class="w-full px-3 py-2 text-sm border border-slate-300 rounded-xl focus:ring-2 focus:ring-emerald-500 focus:outline-none">
                            </div>
                            <div>
                                <label class="block text-xs font-semibold text-slate-600 mb-1">NIP Kepala Sekolah</label>
                                <input type="text" id="cfgNipKepsek" required class="w-full px-3 py-2 text-sm border border-slate-300 rounded-xl focus:ring-2 focus:ring-emerald-500 focus:outline-none">
                            </div>
                        </div>

                        <!-- Pengaturan Jam Terlambat & Threshold Face Match -->
                        <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
                            <div>
                                <label class="block text-xs font-semibold text-slate-600 mb-1">
                                    <i class="fa-regular fa-clock mr-1 text-emerald-600"></i> Jam Batas Terlambat
                                </label>
                                <input type="time" id="cfgJamTerlambat" required class="w-full px-3 py-2 text-sm border border-slate-300 rounded-xl focus:ring-2 focus:ring-emerald-500 focus:outline-none">
                                <p class="text-[10px] text-slate-400 mt-1">Siswa yang melakukan scan setelah jam ini akan otomatis dicatat keterangannya.</p>
                            </div>
                            <div>
                                <label class="block text-xs font-semibold text-slate-600 mb-1">
                                    <i class="fa-solid fa-face-smile mr-1 text-emerald-600"></i> Threshold Face Match (0.0 - 1.0)
                                </label>
                                <input type="number" id="cfgFaceThreshold" min="0" max="1" step="0.01" required placeholder="0.75" class="w-full px-3 py-2 text-sm border border-slate-300 rounded-xl focus:ring-2 focus:ring-emerald-500 focus:outline-none">
                                <p class="text-[10px] text-slate-400 mt-1">Ambang batas tingkat akurasi pencocokan wajah (rekomendasi: 0.70 - 0.85).</p>
                            </div>
                        </div>

                        <div class="pt-4 border-t border-slate-100 flex justify-end">
                            <button type="submit" class="bg-emerald-600 hover:bg-emerald-700 text-white font-medium px-6 py-2.5 rounded-xl text-sm transition shadow-sm flex items-center gap-2">
                                <i class="fa-solid fa-floppy-disk"></i> Simpan Pengaturan
                            </button>
                        </div>
                    </form>
                </div>
            </section>

        </div>
    </main>

    <!-- MODAL: Tambah/Edit Siswa -->
    <div id="modalSiswa" class="fixed inset-0 bg-slate-900/60 backdrop-blur-sm z-50 flex items-center justify-center hidden p-4">
        <div class="bg-white rounded-2xl p-6 w-full max-w-md shadow-2xl relative">
            <h3 id="modalSiswaTitle" class="text-base font-bold text-slate-800 mb-4">Tambah Siswa Baru</h3>
            <form onsubmit="handleSaveStudent(event)" class="space-y-3">
                <input type="hidden" id="studentId">
                <div>
                    <label class="block text-xs font-semibold text-slate-600 mb-1">NIS</label>
                    <input type="text" id="studentNis" required class="w-full px-3 py-2 text-sm border border-slate-300 rounded-xl focus:ring-2 focus:ring-emerald-500 focus:outline-none">
                </div>
                <div>
                    <label class="block text-xs font-semibold text-slate-600 mb-1">NISN</label>
                    <input type="text" id="studentNisn" required class="w-full px-3 py-2 text-sm border border-slate-300 rounded-xl focus:ring-2 focus:ring-emerald-500 focus:outline-none">
                </div>
                <div>
                    <label class="block text-xs font-semibold text-slate-600 mb-1">Nama Lengkap</label>
                    <input type="text" id="studentNama" required class="w-full px-3 py-2 text-sm border border-slate-300 rounded-xl focus:ring-2 focus:ring-emerald-500 focus:outline-none">
                </div>
                <div class="grid grid-cols-2 gap-3">
                    <div>
                        <label class="block text-xs font-semibold text-slate-600 mb-1">Kelas</label>
                        <select id="studentKelasId" required class="w-full px-3 py-2 text-sm border border-slate-300 rounded-xl focus:ring-2 focus:ring-emerald-500 focus:outline-none">
                            <!-- Options -->
                        </select>
                    </div>
                    <div>
                        <label class="block text-xs font-semibold text-slate-600 mb-1">Jenis Kelamin</label>
                        <select id="studentJk" class="w-full px-3 py-2 text-sm border border-slate-300 rounded-xl focus:ring-2 focus:ring-emerald-500 focus:outline-none">
                            <option value="Laki-laki">Laki-laki</option>
                            <option value="Perempuan">Perempuan</option>
                        </select>
                    </div>
                </div>

                <div class="pt-4 flex justify-end gap-2">
                    <button type="button" onclick="closeModalSiswa()" class="px-4 py-2 text-xs font-semibold text-slate-600 bg-slate-100 hover:bg-slate-200 rounded-xl">Batal</button>
                    <button type="submit" class="px-4 py-2 text-xs font-semibold text-white bg-emerald-600 hover:bg-emerald-700 rounded-xl">Simpan</button>
                </div>
            </form>
        </div>
    </div>

    <!-- MODAL: Kartu Presensi & QR Code Siswa -->
    <div id="modalQR" class="fixed inset-0 bg-slate-900/60 backdrop-blur-sm z-50 flex items-center justify-center hidden p-4">
        <div class="bg-white rounded-2xl p-6 w-full max-w-sm text-center shadow-2xl relative space-y-4">
            <button onclick="closeModalQR()" class="absolute top-4 right-4 text-slate-400 hover:text-slate-600"><i class="fa-solid fa-xmark text-lg"></i></button>
            <div id="cardPrintArea" class="border-2 border-emerald-500 rounded-2xl p-5 bg-gradient-to-b from-emerald-50 to-white shadow-sm space-y-3">
                <div class="flex items-center justify-center gap-2">
                    <img id="cardLogo" src="" class="w-8 h-8 object-contain">
                    <p id="cardSchool" class="text-xs font-bold text-slate-800 line-clamp-1"></p>
                </div>
                <hr class="border-slate-200">
                <div id="qrcode" class="flex justify-center my-2 p-2 bg-white rounded-xl shadow-inner w-fit mx-auto border"></div>
                <div>
                    <h4 id="cardNama" class="font-bold text-slate-800 text-sm"></h4>
                    <p id="cardNis" class="text-xs text-slate-500"></p>
                    <span id="cardKelas" class="inline-block mt-1 text-[10px] font-bold bg-emerald-100 text-emerald-800 px-2 py-0.5 rounded-full"></span>
                </div>
            </div>
            <button onclick="printCardOnly()" class="w-full bg-slate-800 hover:bg-slate-900 text-white font-medium py-2 rounded-xl text-xs transition">
                <i class="fa-solid fa-print mr-1"></i> Cetak Kartu Siswa
            </button>
        </div>
    </div>

    <!-- Custom Toast Message -->
    <div id="toast" class="fixed bottom-5 right-5 bg-slate-900 text-white px-4 py-3 rounded-xl shadow-2xl flex items-center gap-3 border border-slate-700 transition transform translate-y-20 opacity-0 z-50 text-xs">
        <i id="toastIcon" class="fa-solid fa-circle-info text-emerald-400"></i>
        <span id="toastMsg">Pesan Notifikasi</span>
    </div>

    <script>
        // State Store
        let state = {
            settings: {
                namaSekolah: 'SD Negeri 01 Nusantara',
                logoSekolah: 'https://cdn-icons-png.flaticon.com/512/2991/2991108.png',
                namaGuru: 'Budi Santoso, S.Pd.',
                nipGuru: '19850101 201001 1 001',
                namaKepsek: 'Drs. H. Ahmad Dahlan, M.Pd.',
                nipKepsek: '19700315 199503 1 002',
                jamTerlambat: '07:15',
                faceThreshold: 0.75
            },
            kelasList: [
                { id: 'KLS01', nama: 'X IPA 1', wali: 'Budi Santoso, S.Pd.' },
                { id: 'KLS02', nama: 'X IPA 2', wali: 'Siti Rahma, S.Pd.' }
            ],
            siswaList: [
                { id: 'SIS001', nis: '1001', nisn: '0051234561', nama: 'Aditya Pratama', kelasId: 'KLS01', namaKelas: 'X IPA 1', jk: 'Laki-laki' },
                { id: 'SIS002', nis: '1002', nisn: '0051234562', nama: 'Anisa Rahmawati', kelasId: 'KLS01', namaKelas: 'X IPA 1', jk: 'Perempuan' },
                { id: 'SIS003', nis: '1003', nisn: '0051234563', nama: 'Bagas Putra', kelasId: 'KLS01', namaKelas: 'X IPA 1', jk: 'Laki-laki' }
            ],
            absensiList: []
        };

        let html5QrCodeScanner = null;
        let myChart = null;

        // Web Audio Beep Sound Effect for Scans
        function playBeep() {
            try {
                const ctx = new (window.AudioContext || window.webkitAudioContext)();
                const osc = ctx.createOscillator();
                const gain = ctx.createGain();
                osc.type = 'sine';
                osc.frequency.value = 880;
                gain.gain.setValueAtTime(0.1, ctx.currentTime);
                osc.connect(gain);
                gain.connect(ctx.destination);
                osc.start();
                osc.stop(ctx.currentTime + 0.15);
            } catch (e) {}
        }

        window.onload = function() {
            // Set default date picker to today
            const today = new Date().toISOString().split('T')[0];
            document.getElementById('absenTanggal').value = today;
            document.getElementById('rekapTanggal').value = today;

            loadInitialData();
        };

        function loadInitialData() {
            if (typeof google !== 'undefined' && google.script && google.script.run) {
                google.script.run
                    .withSuccessHandler(function(response) {
                        try {
                            const res = JSON.parse(response);
                            state.settings = res.settings || state.settings;
                            state.kelasList = res.kelasList || [];
                            state.siswaList = res.siswaList || [];
                            state.absensiList = res.absensiList || [];
                            showToast('Data berhasil disinkronkan dengan Google Spreadsheet', 'success');
                            renderAllViews();
                        } catch(e) {
                            console.error(e);
                        }
                    })
                    .getInitialData();
            } else {
                // Local Storage Fallback for Preview Frame
                const localData = localStorage.getItem('absensi_app_data');
                if (localData) {
                    try { state = JSON.parse(localData); } catch(e){}
                }
                renderAllViews();
            }
        }

        function persistLocal() {
            try {
                localStorage.setItem('absensi_app_data', JSON.stringify(state));
            } catch(e){}
        }

        function renderAllViews() {
            // Update Top Profile & Settings Header
            document.getElementById('appSchoolName').innerText = state.settings.namaSekolah;
            document.getElementById('appLogo').src = state.settings.logoSekolah;
            document.getElementById('topGuruName').innerText = state.settings.namaGuru;
            document.getElementById('topGuruNip').innerText = 'NIP. ' + state.settings.nipGuru;

            // Settings Form inputs
            document.getElementById('cfgNamaSekolah').value = state.settings.namaSekolah;
            document.getElementById('cfgLogoSekolah').value = state.settings.logoSekolah;
            document.getElementById('cfgNamaGuru').value = state.settings.namaGuru;
            document.getElementById('cfgNipGuru').value = state.settings.nipGuru;
            document.getElementById('cfgNamaKepsek').value = state.settings.namaKepsek;
            document.getElementById('cfgNipKepsek').value = state.settings.nipKepsek;
            document.getElementById('cfgJamTerlambat').value = state.settings.jamTerlambat || '07:15';
            document.getElementById('cfgFaceThreshold').value = state.settings.faceThreshold !== undefined ? state.settings.faceThreshold : 0.75;

            // Render sub modules
            renderDashboard();
            renderKelasTable();
            populateClassDropdowns();
            renderSiswaTable();
            loadAbsenSiswaTable();
            generateRekapView();
        }

        function renderDashboard() {
            const today = new Date().toISOString().split('T')[0];
            const todayAbsen = state.absensiList.filter(a => a.tanggal === today);

            let hadir = 0, izin = 0, sakit = 0, alpa = 0;
            todayAbsen.forEach(a => {
                if (a.status === 'H') hadir++;
                else if (a.status === 'I') izin++;
                else if (a.status === 'S') sakit++;
                else if (a.status === 'A') alpa++;
            });

            document.getElementById('dashTotalSiswa').innerText = state.siswaList.length;
            document.getElementById('dashHadir').innerText = hadir;
            document.getElementById('dashIzin').innerText = izin;
            document.getElementById('dashSakit').innerText = sakit;
            document.getElementById('dashAlpa').innerText = alpa;

            // Recent List
            const recentContainer = document.getElementById('recentAttendanceList');
            if (todayAbsen.length === 0) {
                recentContainer.innerHTML = `<p class="text-xs text-slate-400 italic">Belum ada data presensi hari ini.</p>`;
            } else {
                recentContainer.innerHTML = todayAbsen.slice(-5).reverse().map(a => `
                    <div class="flex items-center justify-between p-2 rounded-xl bg-slate-50 border border-slate-100">
                        <div>
                            <p class="font-bold text-xs text-slate-800">${a.namaSiswa}</p>
                            <p class="text-[10px] text-slate-400">${a.namaKelas} • ${a.waktu || '-'}</p>
                        </div>
                        <span class="px-2 py-0.5 rounded-full text-[10px] font-bold ${getStatusBadgeClass(a.status)}">${a.status}</span>
                    </div>
                `).join('');
            }

            // Render Chart
            const ctx = document.getElementById('attendanceChart').getContext('2d');
            if (myChart) myChart.destroy();
            myChart = new Chart(ctx, {
                type: 'bar',
                data: {
                    labels: ['Hadir (H)', 'Izin (I)', 'Sakit (S)', 'Alpa (A)'],
                    datasets: [{
                        label: 'Jumlah Siswa Hari Ini',
                        data: [hadir, izin, sakit, alpa],
                        backgroundColor: ['#22c55e', '#f59e0b', '#6366f1', '#f43f5e'],
                        borderRadius: 8
                    }]
                },
                options: {
                    responsive: true,
                    maintainAspectRatio: false,
                    plugins: { legend: { display: false } },
                    scales: { y: { beginAtZero: true, ticks: { stepSize: 1 } } }
                }
            });
        }

        function getStatusBadgeClass(status) {
            switch(status) {
                case 'H': return 'bg-emerald-100 text-emerald-700';
                case 'I': return 'bg-amber-100 text-amber-700';
                case 'S': return 'bg-indigo-100 text-indigo-700';
                case 'A': return 'bg-rose-100 text-rose-700';
                default: return 'bg-slate-100 text-slate-700';
            }
        }

        // Tab Navigation Switcher
        function switchTab(tabId) {
            document.querySelectorAll('.tab-view').forEach(el => el.classList.add('hidden'));
            document.querySelectorAll('.nav-btn').forEach(el => {
                el.classList.remove('bg-emerald-600', 'text-white');
                el.classList.add('text-slate-300');
            });

            document.getElementById('view-' + tabId).classList.remove('hidden');
            const activeBtn = document.getElementById('nav-' + tabId);
            activeBtn.classList.add('bg-emerald-600', 'text-white');
            activeBtn.classList.remove('text-slate-300');

            // Titles
            const titles = {
                dashboard: ['Dashboard Statistik', 'Ringkasan presensi harian seluruh kelas'],
                kelas: ['Daftar Kelas', 'Kelola data kelas dan wali kelas'],
                siswa: ['Daftar Siswa', 'Kelola biodata siswa & cetak kartu QR Code'],
                absen: ['Presensi & Scan QR', 'Input daftar hadir atau pemindaian kartu otomatis'],
                rekap: ['Rekapitulasi Absensi', 'Laporan harian, bulanan & siap cetak'],
                pengaturan: ['Pengaturan Aplikasi', 'Ubah identitas sekolah, guru & kepala sekolah']
            };
            document.getElementById('pageTitle').innerText = titles[tabId][0];
            document.getElementById('pageSubTitle').innerText = titles[tabId][1];

            // Stop scanner if switching away from absen
            if (tabId !== 'absen' && html5QrCodeScanner) {
                stopScanner();
            }
        }

        // --- DAFTAR KELAS LOGIC ---
        function renderKelasTable() {
            const tbody = document.getElementById('tableKelasBody');
            document.getElementById('countKelas').innerText = state.kelasList.length + ' Kelas';

            if (state.kelasList.length === 0) {
                tbody.innerHTML = `<tr><td colspan="4" class="p-4 text-center text-xs text-slate-400 italic">Belum ada data kelas.</td></tr>`;
                return;
            }

            tbody.innerHTML = state.kelasList.map(k => `
                <tr class="hover:bg-slate-50 transition">
                    <td class="p-3 font-mono text-xs font-semibold text-slate-500">${k.id}</td>
                    <td class="p-3 font-bold text-slate-800">${k.nama}</td>
                    <td class="p-3 text-slate-600">${k.wali}</td>
                    <td class="p-3 text-center">
                        <button onclick="handleDeleteClass('${k.id}')" class="text-rose-600 hover:text-rose-800 p-1.5 rounded-lg hover:bg-rose-50"><i class="fa-solid fa-trash-can"></i></button>
                    </td>
                </tr>
            `).join('');
        }

        function populateClassDropdowns() {
            const options = state.kelasList.map(k => `<option value="${k.id}">${k.nama}</option>`).join('');
            document.getElementById('filterKelasSiswa').innerHTML = '<option value="">-- Semua Kelas --</option>' + options;
            document.getElementById('studentKelasId').innerHTML = options;
            document.getElementById('absenFilterKelas').innerHTML = options;
            document.getElementById('rekapKelas').innerHTML = options;
        }

        function handleSaveClass(e) {
            e.preventDefault();
            const nama = document.getElementById('kelasNama').value.trim();
            const wali = document.getElementById('kelasWali').value.trim();

            const classData = { nama, wali };

            if (typeof google !== 'undefined' && google.script && google.script.run) {
                google.script.run.withSuccessHandler(res => {
                    classData.id = res.id;
                    state.kelasList.push(classData);
                    persistLocal();
                    renderAllViews();
                    document.getElementById('formKelas').reset();
                    showToast(res.message, 'success');
                }).saveClass(classData);
            } else {
                classData.id = 'KLS' + String(Date.now()).slice(-4);
                state.kelasList.push(classData);
                persistLocal();
                renderAllViews();
                document.getElementById('formKelas').reset();
                showToast('Kelas berhasil ditambahkan!', 'success');
            }
        }

        function handleDeleteClass(id) {
            state.kelasList = state.kelasList.filter(k => k.id !== id);
            persistLocal();
            renderAllViews();
            if (typeof google !== 'undefined' && google.script && google.script.run) {
                google.script.run.deleteClass(id);
            }
            showToast('Kelas dihapus', 'info');
        }

        // --- DAFTAR SISWA LOGIC ---
        function renderSiswaTable() {
            const tbody = document.getElementById('tableSiswaBody');
            const kelasFilter = document.getElementById('filterKelasSiswa').value;
            const searchQuery = document.getElementById('searchSiswa').value.toLowerCase();

            let filtered = state.siswaList.filter(s => {
                const matchKelas = !kelasFilter || s.kelasId === kelasFilter;
                const matchSearch = s.nama.toLowerCase().includes(searchQuery) || s.nis.toLowerCase().includes(searchQuery);
                return matchKelas && matchSearch;
            });

            if (filtered.length === 0) {
                tbody.innerHTML = `<tr><td colspan="6" class="p-4 text-center text-xs text-slate-400 italic">Tidak ada data siswa.</td></tr>`;
                return;
            }

            tbody.innerHTML = filtered.map(s => `
                <tr class="hover:bg-slate-50 transition">
                    <td class="p-3">
                        <span class="font-bold text-slate-800">${s.nis}</span>
                        <span class="block text-[10px] text-slate-400">NISN: ${s.nisn}</span>
                    </td>
                    <td class="p-3 font-semibold text-slate-800">${s.nama}</td>
                    <td class="p-3 text-slate-600"><span class="bg-slate-100 text-slate-700 px-2 py-0.5 rounded-full text-xs">${s.namaKelas}</span></td>
                    <td class="p-3 text-xs text-slate-500">${s.jk}</td>
                    <td class="p-3 text-center">
                        <button onclick="showStudentQR('${s.id}')" class="bg-emerald-50 text-emerald-700 hover:bg-emerald-100 px-3 py-1 rounded-xl text-xs font-semibold border border-emerald-200 transition">
                            <i class="fa-solid fa-qrcode mr-1"></i> Lihat Kartu
                        </button>
                    </td>
                    <td class="p-3 text-center space-x-1">
                        <button onclick="editSiswa('${s.id}')" class="text-blue-600 hover:text-blue-800 p-1.5 rounded-lg hover:bg-blue-50"><i class="fa-solid fa-pen-to-square"></i></button>
                        <button onclick="deleteSiswa('${s.id}')" class="text-rose-600 hover:text-rose-800 p-1.5 rounded-lg hover:bg-rose-50"><i class="fa-solid fa-trash-can"></i></button>
                    </td>
                </tr>
            `).join('');
        }

        function openModalSiswa(siswa = null) {
            document.getElementById('modalSiswa').classList.remove('hidden');
            if (siswa) {
                document.getElementById('modalSiswaTitle').innerText = 'Edit Data Siswa';
                document.getElementById('studentId').value = siswa.id;
                document.getElementById('studentNis').value = siswa.nis;
                document.getElementById('studentNisn').value = siswa.nisn;
                document.getElementById('studentNama').value = siswa.nama;
                document.getElementById('studentKelasId').value = siswa.kelasId;
                document.getElementById('studentJk').value = siswa.jk;
            } else {
                document.getElementById('modalSiswaTitle').innerText = 'Tambah Siswa Baru';
                document.getElementById('studentId').value = '';
                document.getElementById('studentNis').value = '';
                document.getElementById('studentNisn').value = '';
                document.getElementById('studentNama').value = '';
            }
        }

        function closeModalSiswa() {
            document.getElementById('modalSiswa').classList.add('hidden');
        }

        function handleSaveStudent(e) {
            e.preventDefault();
            const id = document.getElementById('studentId').value;
            const nis = document.getElementById('studentNis').value.trim();
            const nisn = document.getElementById('studentNisn').value.trim();
            const nama = document.getElementById('studentNama').value.trim();
            const kelasId = document.getElementById('studentKelasId').value;
            const jk = document.getElementById('studentJk').value;

            const kelasObj = state.kelasList.find(k => k.id === kelasId);
            const namaKelas = kelasObj ? kelasObj.nama : '';

            const studentData = { id, nis, nisn, nama, kelasId, namaKelas, jk };

            if (typeof google !== 'undefined' && google.script && google.script.run) {
                google.script.run.withSuccessHandler(res => {
                    studentData.id = res.id;
                    const idx = state.siswaList.findIndex(s => s.id === res.id);
                    if (idx >= 0) state.siswaList[idx] = studentData;
                    else state.siswaList.push(studentData);
                    persistLocal();
                    renderAllViews();
                    closeModalSiswa();
                    showToast(res.message, 'success');
                }).saveStudent(studentData);
            } else {
                if (id) {
                    const idx = state.siswaList.findIndex(s => s.id === id);
                    if (idx >= 0) state.siswaList[idx] = studentData;
                } else {
                    studentData.id = 'SIS' + String(Date.now()).slice(-5);
                    state.siswaList.push(studentData);
                }
                persistLocal();
                renderAllViews();
                closeModalSiswa();
                showToast('Data siswa disimpan!', 'success');
            }
        }

        function editSiswa(id) {
            const siswa = state.siswaList.find(s => s.id === id);
            if (siswa) openModalSiswa(siswa);
        }

        function deleteSiswa(id) {
            state.siswaList = state.siswaList.filter(s => s.id !== id);
            persistLocal();
            renderAllViews();
            if (typeof google !== 'undefined' && google.script && google.script.run) {
                google.script.run.deleteStudent(id);
            }
            showToast('Siswa dihapus', 'info');
        }

        function showStudentQR(id) {
            const siswa = state.siswaList.find(s => s.id === id);
            if (!siswa) return;

            document.getElementById('cardSchool').innerText = state.settings.namaSekolah;
            document.getElementById('cardLogo').src = state.settings.logoSekolah;
            document.getElementById('cardNama').innerText = siswa.nama;
            document.getElementById('cardNis').innerText = 'NIS: ' + siswa.nis;
            document.getElementById('cardKelas').innerText = siswa.namaKelas;

            const qrContainer = document.getElementById('qrcode');
            qrContainer.innerHTML = '';
            new QRCode(qrContainer, {
                text: siswa.id,
                width: 140,
                height: 140,
                colorDark : "#0f172a",
                colorLight : "#ffffff",
                correctLevel : QRCode.CorrectLevel.H
            });

            document.getElementById('modalQR').classList.remove('hidden');
        }

        function closeModalQR() {
            document.getElementById('modalQR').classList.add('hidden');
        }

        function printCardOnly() {
            const cardContent = document.getElementById('cardPrintArea').outerHTML;
            const win = window.open('', '_blank');
            win.document.write(`
                <html>
                <head>
                    <title>Cetak Kartu Siswa</title>
                    <script src="https://cdn.tailwindcss.com"><\/script>
                </head>
                <body class="p-8 flex justify-center items-center">
                    ${cardContent}
                    <script>
                        setTimeout(() => { window.print(); window.close(); }, 500);
                    <\/script>
                </body>
                </html>
            `);
            win.document.close();
        }

        // --- ABSENSI LOGIC & QR SCANNER ---
        function setAbsenMode(mode) {
            if (mode === 'scan') {
                document.getElementById('containerManual').classList.add('hidden');
                document.getElementById('containerScanner').classList.remove('hidden');
                document.getElementById('btnModeScan').classList.add('bg-white', 'text-emerald-700', 'shadow-sm');
                document.getElementById('btnModeScan').classList.remove('text-slate-600');
                document.getElementById('btnModeManual').classList.remove('bg-white', 'text-emerald-700', 'shadow-sm');
                document.getElementById('btnModeManual').classList.add('text-slate-600');
                startScanner();
            } else {
                document.getElementById('containerScanner').classList.add('hidden');
                document.getElementById('containerManual').classList.remove('hidden');
                document.getElementById('btnModeManual').classList.add('bg-white', 'text-emerald-700', 'shadow-sm');
                document.getElementById('btnModeManual').classList.remove('text-slate-600');
                document.getElementById('btnModeScan').classList.remove('bg-white', 'text-emerald-700', 'shadow-sm');
                document.getElementById('btnModeScan').classList.add('text-slate-600');
                stopScanner();
            }
        }

        function startScanner() {
            if (html5QrCodeScanner) return;
            html5QrCodeScanner = new Html5Qrcode("reader");
            html5QrCodeScanner.start(
                { facingMode: "environment" },
                { fps: 10, qrbox: { width: 220, height: 220 } },
                onScanSuccess,
                onScanError
            ).catch(err => {
                document.getElementById('scanFeedback').innerText = "Kamera tidak dapat diakses atau diblokir izinnya.";
            });
        }

        function stopScanner() {
            if (html5QrCodeScanner) {
                html5QrCodeScanner.stop().then(() => {
                    html5QrCodeScanner.clear();
                    html5QrCodeScanner = null;
                }).catch(err => console.error(err));
            }
        }

        let lastScannedId = '';
        let lastScanTime = 0;

        function onScanSuccess(decodedText, decodedResult) {
            const now = Date.now();
            if (decodedText === lastScannedId && (now - lastScanTime) < 3000) return; // Prevent spam scan
            lastScannedId = decodedText;
            lastScanTime = now;

            const student = state.siswaList.find(s => s.id === decodedText || s.nis === decodedText);
            if (!student) {
                document.getElementById('scanFeedback').innerHTML = `<span class="text-rose-400 font-bold"><i class="fa-solid fa-circle-xmark"></i> QR Code Tidak Dikenali!</span>`;
                return;
            }

            playBeep();
            const today = document.getElementById('absenTanggal').value;
            const timeStr = new Date().toLocaleTimeString('id-ID', { hour: '2-digit', minute: '2-digit' });

            // Cek keterlambatan berdasarkan jamTerlambat
            let isLate = false;
            if (state.settings.jamTerlambat) {
                const nowHHMM = new Date().toTimeString().substring(0, 5);
                if (nowHHMM > state.settings.jamTerlambat) {
                    isLate = true;
                }
            }

            // Record attendance auto "Hadir"
            const newRecord = {
                tanggal: today,
                waktu: timeStr,
                siswaId: student.id,
                nis: student.nis,
                namaSiswa: student.nama,
                kelasId: student.kelasId,
                namaKelas: student.namaKelas,
                status: 'H',
                keterangan: isLate ? `Terlambat (Batas ${state.settings.jamTerlambat})` : 'Scan QR Barcode'
            };

            // Remove existing record today if re-scanned
            state.absensiList = state.absensiList.filter(a => !(a.siswaId === student.id && a.tanggal === today));
            state.absensiList.push(newRecord);
            persistLocal();

            if (typeof google !== 'undefined' && google.script && google.script.run) {
                google.script.run.saveAttendanceBatch([newRecord]);
            }

            document.getElementById('scanFeedback').innerHTML = `
                <div class="text-center">
                    <p class="text-emerald-400 font-bold text-sm"><i class="fa-solid fa-circle-check"></i> ABSEN BERHASIL!</p>
                    <p class="text-white font-semibold text-xs mt-1">${student.nama} (${student.namaKelas})</p>
                    <p class="text-[10px] text-slate-400">${today} Pukul ${timeStr} • HADIR</p>
                </div>
            `;

            renderDashboard();
            loadAbsenSiswaTable();
        }

        function onScanError(error) {
            // Silence scan errors during continuous scanning
        }

        function loadAbsenSiswaTable() {
            const tbody = document.getElementById('tableAbsenBody');
            const kelasId = document.getElementById('absenFilterKelas').value;
            const tanggal = document.getElementById('absenTanggal').value;

            const studentsInClass = state.siswaList.filter(s => s.kelasId === kelasId);

            if (studentsInClass.length === 0) {
                tbody.innerHTML = `<tr><td colspan="5" class="p-4 text-center text-xs text-slate-400 italic">Pilih kelas yang memiliki siswa terdaftar.</td></tr>`;
                return;
            }

            tbody.innerHTML = studentsInClass.map((s, idx) => {
                const existing = state.absensiList.find(a => a.siswaId === s.id && a.tanggal === tanggal);
                const currentStatus = existing ? existing.status : 'H';
                const currentKet = existing ? existing.keterangan : '';

                return `
                    <tr class="hover:bg-slate-50 transition" data-siswa-id="${s.id}">
                        <td class="p-3 text-xs text-slate-500">${idx + 1}</td>
                        <td class="p-3 font-mono text-xs text-slate-600">${s.nis}</td>
                        <td class="p-3 font-semibold text-slate-800">${s.nama}</td>
                        <td class="p-3 text-center">
                            <div class="inline-flex rounded-xl bg-slate-100 p-1 gap-1">
                                <label class="cursor-pointer">
                                    <input type="radio" name="status_${s.id}" value="H" ${currentStatus==='H'?'checked':''} class="peer hidden">
                                    <span class="px-2.5 py-1 text-xs font-bold rounded-lg block peer-checked:bg-emerald-600 peer-checked:text-white text-slate-600 hover:text-slate-800">H</span>
                                </label>
                                <label class="cursor-pointer">
                                    <input type="radio" name="status_${s.id}" value="I" ${currentStatus==='I'?'checked':''} class="peer hidden">
                                    <span class="px-2.5 py-1 text-xs font-bold rounded-lg block peer-checked:bg-amber-500 peer-checked:text-white text-slate-600 hover:text-slate-800">I</span>
                                </label>
                                <label class="cursor-pointer">
                                    <input type="radio" name="status_${s.id}" value="S" ${currentStatus==='S'?'checked':''} class="peer hidden">
                                    <span class="px-2.5 py-1 text-xs font-bold rounded-lg block peer-checked:bg-indigo-600 peer-checked:text-white text-slate-600 hover:text-slate-800">S</span>
                                </label>
                                <label class="cursor-pointer">
                                    <input type="radio" name="status_${s.id}" value="A" ${currentStatus==='A'?'checked':''} class="peer hidden">
                                    <span class="px-2.5 py-1 text-xs font-bold rounded-lg block peer-checked:bg-rose-600 peer-checked:text-white text-slate-600 hover:text-slate-800">A</span>
                                </label>
                            </div>
                        </td>
                        <td class="p-3">
                            <input type="text" id="ket_${s.id}" value="${currentKet}" placeholder="Catatan (Opsional)" class="w-full px-2.5 py-1 text-xs border border-slate-300 rounded-lg focus:outline-none focus:ring-1 focus:ring-emerald-500">
                        </td>
                    </tr>
                `;
            }).join('');
        }

        function submitAttendanceBatch() {
            const kelasId = document.getElementById('absenFilterKelas').value;
            const tanggal = document.getElementById('absenTanggal').value;
            const studentsInClass = state.siswaList.filter(s => s.kelasId === kelasId);

            if (studentsInClass.length === 0) return;

            const timeStr = new Date().toLocaleTimeString('id-ID', { hour: '2-digit', minute: '2-digit' });
            const batchRecords = [];

            studentsInClass.forEach(s => {
                const statusRadio = document.querySelector(`input[name="status_${s.id}"]:checked`);
                const status = statusRadio ? statusRadio.value : 'H';
                const keterangan = document.getElementById(`ket_${s.id}`).value;

                const rec = {
                    tanggal: tanggal,
                    waktu: timeStr,
                    siswaId: s.id,
                    nis: s.nis,
                    namaSiswa: s.nama,
                    kelasId: s.kelasId,
                    namaKelas: s.namaKelas,
                    status: status,
                    keterangan: keterangan
                };

                // Clear duplicate
                state.absensiList = state.absensiList.filter(a => !(a.siswaId === s.id && a.tanggal === tanggal));
                state.absensiList.push(rec);
                batchRecords.push(rec);
            });

            persistLocal();

            if (typeof google !== 'undefined' && google.script && google.script.run) {
                google.script.run.withSuccessHandler(res => {
                    showToast(res.message, 'success');
                }).saveAttendanceBatch(batchRecords);
            } else {
                showToast('Presensi kelas berhasil disimpan!', 'success');
            }

            renderDashboard();
        }

        // --- REKAP ABSEN LOGIC ---
        function toggleRekapType() {
            const type = document.getElementById('rekapType').value;
            if (type === 'harian') {
                document.getElementById('wrapRekapTgl').classList.remove('hidden');
                document.getElementById('wrapRekapBulan').classList.add('hidden');
            } else {
                document.getElementById('wrapRekapTgl').classList.add('hidden');
                document.getElementById('wrapRekapBulan').classList.remove('hidden');
            }
        }

        function generateRekapView() {
            const type = document.getElementById('rekapType').value;
            const kelasId = document.getElementById('rekapKelas').value;
            const table = document.getElementById('tableRekap');

            const kelasObj = state.kelasList.find(k => k.id === kelasId);
            const namaKelas = kelasObj ? kelasObj.nama : 'Semua Kelas';
            const studentsInClass = state.siswaList.filter(s => !kelasId || s.kelasId === kelasId);

            if (type === 'harian') {
                const tanggal = document.getElementById('rekapTanggal').value;
                document.getElementById('rekapTitleText').innerText = `REKAPITULASI PRESENSI HARIAN (${namaKelas})`;
                document.getElementById('rekapSubText').innerText = `Tanggal: ${tanggal}`;

                let html = `
                    <thead>
                        <tr class="bg-slate-100 text-slate-700 uppercase font-bold border-b border-slate-200">
                            <th class="p-3 border">No</th>
                            <th class="p-3 border">NIS</th>
                            <th class="p-3 border">Nama Siswa</th>
                            <th class="p-3 border text-center">Status</th>
                            <th class="p-3 border">Waktu Scan</th>
                            <th class="p-3 border">Keterangan</th>
                        </tr>
                    </thead>
                    <tbody class="divide-y divide-slate-200">
                `;

                studentsInClass.forEach((s, idx) => {
                    const rec = state.absensiList.find(a => a.siswaId === s.id && a.tanggal === tanggal);
                    const status = rec ? rec.status : 'Belum Absen';
                    const waktu = rec ? (rec.waktu || '-') : '-';
                    const ket = rec ? rec.keterangan : '-';

                    html += `
                        <tr>
                            <td class="p-2.5 border text-center font-medium">${idx + 1}</td>
                            <td class="p-2.5 border font-mono">${s.nis}</td>
                            <td class="p-2.5 border font-semibold">${s.nama}</td>
                            <td class="p-2.5 border text-center"><span class="px-2 py-0.5 rounded-full text-xs font-bold ${getStatusBadgeClass(status)}">${status}</span></td>
                            <td class="p-2.5 border text-center">${waktu}</td>
                            <td class="p-2.5 border text-slate-500">${ket}</td>
                        </tr>
                    `;
                });

                html += `</tbody>`;
                table.innerHTML = html;

            } else {
                // Monthly recap
                const bulan = document.getElementById('rekapBulan').value;
                const tahun = document.getElementById('rekapTahun').value;
                const daysInMonth = new Date(tahun, parseInt(bulan), 0).getDate();

                document.getElementById('rekapTitleText').innerText = `REKAPITULASI PRESENSI BULANAN (${namaKelas})`;
                document.getElementById('rekapSubText').innerText = `Bulan: ${bulan} / Tahun: ${tahun}`;

                let html = `
                    <thead>
                        <tr class="bg-slate-100 text-slate-700 uppercase font-bold text-[10px] border-b border-slate-200">
                            <th class="p-1 border text-center" rowspan="2">No</th>
                            <th class="p-1 border text-left" rowspan="2">Nama Siswa</th>
                            <th class="p-1 border text-center" colspan="${daysInMonth}">Tanggal</th>
                            <th class="p-1 border text-center" colspan="4">Total</th>
                        </tr>
                        <tr class="bg-slate-50 text-slate-600 text-[9px] border-b border-slate-200">
                            ${Array.from({length: daysInMonth}, (_, i) => `<th class="p-1 border text-center w-6">${i+1}</th>`).join('')}
                            <th class="p-1 border bg-emerald-100 text-emerald-800">H</th>
                            <th class="p-1 border bg-amber-100 text-amber-800">I</th>
                            <th class="p-1 border bg-indigo-100 text-indigo-800">S</th>
                            <th class="p-1 border bg-rose-100 text-rose-800">A</th>
                        </tr>
                    </thead>
                    <tbody class="divide-y divide-slate-200 text-[11px]">
                `;

                studentsInClass.forEach((s, idx) => {
                    let totalH = 0, totalI = 0, totalS = 0, totalA = 0;
                    let dayCells = '';

                    for (let d = 1; d <= daysInMonth; d++) {
                        const dayStr = String(d).padStart(2, '0');
                        const fullDate = `${tahun}-${bulan}-${dayStr}`;
                        const rec = state.absensiList.find(a => a.siswaId === s.id && a.tanggal === fullDate);
                        const st = rec ? rec.status : '';

                        if (st === 'H') totalH++;
                        else if (st === 'I') totalI++;
                        else if (st === 'S') totalS++;
                        else if (st === 'A') totalA++;

                        dayCells += `<td class="p-1 border text-center font-bold text-[10px]">${st}</td>`;
                    }

                    html += `
                        <tr>
                            <td class="p-1 border text-center font-medium">${idx + 1}</td>
                            <td class="p-1.5 border font-medium truncate max-w-[120px]">${s.nama}</td>
                            ${dayCells}
                            <td class="p-1 border text-center font-bold text-emerald-700 bg-emerald-50">${totalH}</td>
                            <td class="p-1 border text-center font-bold text-amber-700 bg-amber-50">${totalI}</td>
                            <td class="p-1 border text-center font-bold text-indigo-700 bg-indigo-50">${totalS}</td>
                            <td class="p-1 border text-center font-bold text-rose-700 bg-rose-50">${totalA}</td>
                        </tr>
                    `;
                });

                html += `</tbody>`;
                table.innerHTML = html;
            }
        }

        function printRekapNewTab() {
            const title = document.getElementById('rekapTitleText').innerText;
            const subtitle = document.getElementById('rekapSubText').innerText;
            const tableContent = document.getElementById('tableRekap').outerHTML;

            const printWindow = window.open('', '_blank');
            printWindow.document.write(`
                <!DOCTYPE html>
                <html>
                <head>
                    <title>${title}</title>
                    <script src="https://cdn.tailwindcss.com"><\/script>
                    <style>
                        @media print {
                            body { -webkit-print-color-adjust: exact; font-size: 10pt; }
                            @page { size: landscape; margin: 1cm; }
                        }
                    </style>
                </head>
                <body class="p-8 text-slate-800 bg-white">
                    <!-- Kop Surat -->
                    <div class="flex items-center justify-between pb-4 mb-6 border-b-2 border-slate-900">
                        <div class="flex items-center gap-4">
                            <img src="${state.settings.logoSekolah}" class="w-16 h-16 object-contain">
                            <div>
                                <h1 class="text-xl font-bold uppercase tracking-wide text-slate-900">${state.settings.namaSekolah}</h1>
                                <p class="text-xs text-slate-600">Laporan Resmi Presensi Kehadiran Siswa</p>
                            </div>
                        </div>
                    </div>

                    <div class="text-center mb-6">
                        <h2 class="text-lg font-bold text-slate-900 uppercase">${title}</h2>
                        <p class="text-xs text-slate-600 font-medium">${subtitle}</p>
                    </div>

                    <div class="overflow-x-auto mb-12">
                        ${tableContent}
                    </div>

                    <!-- Tanda Tangan -->
                    <div class="grid grid-cols-2 gap-8 text-xs pt-8 border-t border-slate-200">
                        <div class="text-center space-y-16">
                            <div>
                                <p>Mengetahui,</p>
                                <p class="font-bold">Kepala Sekolah</p>
                            </div>
                            <div>
                                <p class="font-bold underline">${state.settings.namaKepsek}</p>
                                <p>NIP. ${state.settings.nipKepsek}</p>
                            </div>
                        </div>

                        <div class="text-center space-y-16">
                            <div>
                                <p>Guru / Wali Kelas,</p>
                            </div>
                            <div>
                                <p class="font-bold underline">${state.settings.namaGuru}</p>
                                <p>NIP. ${state.settings.nipGuru}</p>
                            </div>
                        </div>
                    </div>

                    <script>
                        setTimeout(() => {
                            window.print();
                        }, 600);
                    <\/script>
                </body>
                </html>
            `);
            printWindow.document.close();
        }

        // --- PENGATURAN LOGIC ---
        function handleSaveSettings(e) {
            e.preventDefault();
            const newSettings = {
                namaSekolah: document.getElementById('cfgNamaSekolah').value.trim(),
                logoSekolah: document.getElementById('cfgLogoSekolah').value.trim(),
                namaGuru: document.getElementById('cfgNamaGuru').value.trim(),
                nipGuru: document.getElementById('cfgNipGuru').value.trim(),
                namaKepsek: document.getElementById('cfgNamaKepsek').value.trim(),
                nipKepsek: document.getElementById('cfgNipKepsek').value.trim(),
                jamTerlambat: document.getElementById('cfgJamTerlambat').value,
                faceThreshold: parseFloat(document.getElementById('cfgFaceThreshold').value) || 0.75
            };

            state.settings = newSettings;
            persistLocal();
            renderAllViews();

            if (typeof google !== 'undefined' && google.script && google.script.run) {
                google.script.run.withSuccessHandler(res => {
                    showToast(res.message, 'success');
                }).saveSettings(newSettings);
            } else {
                showToast('Pengaturan berhasil disimpan!', 'success');
            }
        }

        // UI Toast Helper
        function showToast(msg, type = 'info') {
            const toast = document.getElementById('toast');
            document.getElementById('toastMsg').innerText = msg;
            toast.classList.remove('translate-y-20', 'opacity-0');
            setTimeout(() => {
                toast.classList.add('translate-y-20', 'opacity-0');
            }, 3000);
        }
    </script>
</body>
</html>
