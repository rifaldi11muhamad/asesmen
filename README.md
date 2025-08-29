<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Sistem Penilaian Siswa</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
    <link href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css" rel="stylesheet">
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700&display=swap');
        body { font-family: 'Inter', sans-serif; }
        .gradient-bg { background: linear-gradient(135deg, #667eea 0%, #764ba2 100%); }
        .card-shadow { box-shadow: 0 10px 25px rgba(0,0,0,0.1); }
        .hover-scale { transition: transform 0.2s; }
        .hover-scale:hover { transform: scale(1.02); }
        .notification {
            position: fixed;
            top: 20px;
            right: 20px;
            padding: 12px 24px;
            border-radius: 8px;
            color: white;
            font-weight: 500;
            z-index: 1000;
            transform: translateX(400px);
            transition: transform 0.3s ease;
        }
        .notification.show { transform: translateX(0); }
        .notification.success { background-color: #10b981; }
        .notification.error { background-color: #ef4444; }
        .notification.warning { background-color: #f59e0b; }
        .auto-save-indicator {
            position: fixed;
            bottom: 20px;
            right: 20px;
            background: rgba(16, 185, 129, 0.9);
            color: white;
            padding: 8px 16px;
            border-radius: 20px;
            font-size: 12px;
            z-index: 1000;
            opacity: 0;
            transition: opacity 0.3s ease;
            box-shadow: 0 4px 12px rgba(0,0,0,0.15);
        }
        .auto-save-indicator.show { opacity: 1; }
        .pulse-dot {
            width: 8px;
            height: 8px;
            background: #10b981;
            border-radius: 50%;
            animation: pulse 2s infinite;
            margin-right: 8px;
        }
        @keyframes pulse {
            0% { transform: scale(0.95); box-shadow: 0 0 0 0 rgba(16, 185, 129, 0.7); }
            70% { transform: scale(1); box-shadow: 0 0 0 10px rgba(16, 185, 129, 0); }
            100% { transform: scale(0.95); box-shadow: 0 0 0 0 rgba(16, 185, 129, 0); }
        }
    </style>
</head>
<body class="bg-gray-50">
    <!-- Notification -->
    <div id="notification" class="notification"></div>
    
    <!-- Auto-save Indicator -->
    <div id="autoSaveIndicator" class="auto-save-indicator">
        <div class="flex items-center">
            <div class="pulse-dot"></div>
            <span>Data tersimpan otomatis</span>
        </div>
    </div>

    <!-- Header -->
    <header class="gradient-bg text-white shadow-lg">
        <div class="container mx-auto px-6 py-4">
            <div class="flex items-center justify-between">
                <div class="flex items-center space-x-4">
                    <div class="w-12 h-12 bg-white rounded-full flex items-center justify-center">
                        <i class="fas fa-graduation-cap text-blue-600 text-xl"></i>
                    </div>
                    <div>
                        <h1 class="text-2xl font-bold" id="schoolName">SMA Negeri 1 Jakarta</h1>
                        <p class="text-blue-100 text-sm">Sistem Penilaian Siswa Digital</p>
                    </div>
                </div>
                <button onclick="openSettings()" class="bg-white bg-opacity-20 hover:bg-opacity-30 px-4 py-2 rounded-lg transition-all">
                    <i class="fas fa-cog mr-2"></i>Pengaturan
                </button>
            </div>
        </div>
    </header>

    <!-- Navigation -->
    <nav class="bg-white shadow-md">
        <div class="container mx-auto px-6">
            <div class="flex space-x-8">
                <button onclick="showTab('dashboard')" class="nav-btn py-4 px-2 border-b-2 border-blue-500 text-blue-600 font-medium">
                    <i class="fas fa-tachometer-alt mr-2"></i>Dashboard
                </button>
                <button onclick="showTab('students')" class="nav-btn py-4 px-2 border-b-2 border-transparent text-gray-500 hover:text-gray-700">
                    <i class="fas fa-users mr-2"></i>Data Siswa
                </button>
                <button onclick="showTab('grades')" class="nav-btn py-4 px-2 border-b-2 border-transparent text-gray-500 hover:text-gray-700">
                    <i class="fas fa-chart-line mr-2"></i>Penilaian
                </button>
                <button onclick="showTab('reports')" class="nav-btn py-4 px-2 border-b-2 border-transparent text-gray-500 hover:text-gray-700">
                    <i class="fas fa-file-alt mr-2"></i>Laporan
                </button>
                <button onclick="showTab('management')" class="nav-btn py-4 px-2 border-b-2 border-transparent text-gray-500 hover:text-gray-700">
                    <i class="fas fa-cogs mr-2"></i>Manajemen
                </button>
            </div>
        </div>
    </nav>

    <!-- Main Content -->
    <main class="container mx-auto px-6 py-8">
        <!-- Dashboard Tab -->
        <div id="dashboard" class="tab-content">
            <!-- Auto-save Status -->
            <div class="bg-green-50 border border-green-200 rounded-lg p-4 mb-6">
                <div class="flex items-center">
                    <div class="pulse-dot"></div>
                    <div>
                        <h4 class="text-green-800 font-medium">Auto-Save Aktif</h4>
                        <p class="text-green-600 text-sm">Data akan tersimpan otomatis setiap 30 detik dan saat ada perubahan</p>
                    </div>
                </div>
            </div>

            <!-- Statistics Cards -->
            <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-6 mb-8">
                <div class="bg-white p-6 rounded-xl card-shadow hover-scale">
                    <div class="flex items-center">
                        <div class="p-3 bg-blue-100 rounded-full">
                            <i class="fas fa-users text-blue-600 text-xl"></i>
                        </div>
                        <div class="ml-4">
                            <p class="text-gray-500 text-sm">Total Siswa</p>
                            <p class="text-2xl font-bold text-gray-800" id="totalStudents">0</p>
                        </div>
                    </div>
                </div>
                <div class="bg-white p-6 rounded-xl card-shadow hover-scale">
                    <div class="flex items-center">
                        <div class="p-3 bg-green-100 rounded-full">
                            <i class="fas fa-chart-line text-green-600 text-xl"></i>
                        </div>
                        <div class="ml-4">
                            <p class="text-gray-500 text-sm">Rata-rata Kelas</p>
                            <p class="text-2xl font-bold text-gray-800" id="classAverage">0</p>
                        </div>
                    </div>
                </div>
                <div class="bg-white p-6 rounded-xl card-shadow hover-scale">
                    <div class="flex items-center">
                        <div class="p-3 bg-yellow-100 rounded-full">
                            <i class="fas fa-trophy text-yellow-600 text-xl"></i>
                        </div>
                        <div class="ml-4">
                            <p class="text-gray-500 text-sm">Nilai Tertinggi</p>
                            <p class="text-2xl font-bold text-gray-800" id="highestGrade">0</p>
                        </div>
                    </div>
                </div>
                <div class="bg-white p-6 rounded-xl card-shadow hover-scale">
                    <div class="flex items-center">
                        <div class="p-3 bg-red-100 rounded-full">
                            <i class="fas fa-exclamation-triangle text-red-600 text-xl"></i>
                        </div>
                        <div class="ml-4">
                            <p class="text-gray-500 text-sm">Perlu Perhatian</p>
                            <p class="text-2xl font-bold text-gray-800" id="needAttention">0</p>
                        </div>
                    </div>
                </div>
            </div>

            <!-- Top Students -->
            <div class="bg-white p-6 rounded-xl card-shadow mb-8">
                <h3 class="text-xl font-bold text-gray-800 mb-4">🏆 Top 10 Siswa Terbaik</h3>
                <div id="topStudentsList" class="space-y-3">
                    <!-- Top students will be populated here -->
                </div>
            </div>

            <!-- Quick Actions -->
            <div class="grid grid-cols-1 md:grid-cols-3 gap-6">
                <div class="bg-white p-6 rounded-xl card-shadow">
                    <h4 class="text-lg font-semibold text-gray-800 mb-4">Aksi Cepat</h4>
                    <div class="space-y-3">
                        <button onclick="showTab('students')" class="w-full bg-blue-600 hover:bg-blue-700 text-white py-2 px-4 rounded-lg transition-colors">
                            <i class="fas fa-plus mr-2"></i>Tambah Siswa
                        </button>
                        <button onclick="showTab('grades')" class="w-full bg-green-600 hover:bg-green-700 text-white py-2 px-4 rounded-lg transition-colors">
                            <i class="fas fa-edit mr-2"></i>Input Nilai
                        </button>
                        <button onclick="showTab('reports')" class="w-full bg-purple-600 hover:bg-purple-700 text-white py-2 px-4 rounded-lg transition-colors">
                            <i class="fas fa-download mr-2"></i>Download Laporan
                        </button>
                    </div>
                </div>
                <div class="bg-white p-6 rounded-xl card-shadow">
                    <h4 class="text-lg font-semibold text-gray-800 mb-4">Statistik Nilai</h4>
                    <div class="space-y-3">
                        <div class="flex justify-between">
                            <span class="text-gray-600">Sangat Baik (90-100)</span>
                            <span class="font-semibold" id="excellentCount">0</span>
                        </div>
                        <div class="flex justify-between">
                            <span class="text-gray-600">Baik (80-89)</span>
                            <span class="font-semibold" id="goodCount">0</span>
                        </div>
                        <div class="flex justify-between">
                            <span class="text-gray-600">Cukup (70-79)</span>
                            <span class="font-semibold" id="fairCount">0</span>
                        </div>
                        <div class="flex justify-between">
                            <span class="text-gray-600">Kurang (<70)</span>
                            <span class="font-semibold" id="poorCount">0</span>
                        </div>
                    </div>
                </div>
                <div class="bg-white p-6 rounded-xl card-shadow">
                    <h4 class="text-lg font-semibold text-gray-800 mb-4">Info Sistem</h4>
                    <div class="space-y-3 text-sm text-gray-600">
                        <div>Total Kelas: <span class="font-semibold" id="totalClasses">0</span></div>
                        <div>Total Guru: <span class="font-semibold" id="totalTeachers">0</span></div>
                        <div>Mata Pelajaran: <span class="font-semibold" id="totalSubjects">0</span></div>
                        <div>Data Tersimpan: <span class="font-semibold text-green-600">✓ Aktif</span></div>
                        <div>Auto-Save: <span class="font-semibold text-blue-600">✓ Setiap 30 detik</span></div>
                        <div>Terakhir Disimpan: <span class="font-semibold text-purple-600" id="lastSavedTime">-</span></div>
                    </div>
                </div>
            </div>
        </div>

        <!-- Students Tab -->
        <div id="students" class="tab-content hidden">
            <div class="bg-white rounded-xl card-shadow">
                <div class="p-6 border-b border-gray-200">
                    <div class="flex flex-col sm:flex-row justify-between items-start sm:items-center space-y-4 sm:space-y-0">
                        <h3 class="text-xl font-bold text-gray-800">Data Siswa</h3>
                        <div class="flex flex-wrap gap-3">
                            <input type="file" id="excelFile" accept=".xlsx,.xls" class="hidden" onchange="importExcel()">
                            <button onclick="downloadExcelTemplate()" class="bg-purple-600 hover:bg-purple-700 text-white px-4 py-2 rounded-lg transition-colors">
                                <i class="fas fa-download mr-2"></i>Download Template
                            </button>
                            <button onclick="document.getElementById('excelFile').click()" class="bg-green-600 hover:bg-green-700 text-white px-4 py-2 rounded-lg transition-colors">
                                <i class="fas fa-file-excel mr-2"></i>Import Excel
                            </button>
                            <button onclick="openAddStudentModal()" class="bg-blue-600 hover:bg-blue-700 text-white px-4 py-2 rounded-lg transition-colors">
                                <i class="fas fa-plus mr-2"></i>Tambah Siswa
                            </button>
                        </div>
                    </div>
                    
                    <!-- Bulk Actions -->
                    <div id="studentBulkActions" class="hidden bg-blue-50 p-4 rounded-lg mb-4 mt-4">
                        <div class="flex flex-wrap items-center gap-3">
                            <span class="text-sm font-medium text-blue-800">
                                <span id="selectedStudentsCount">0</span> siswa dipilih
                            </span>
                            <button onclick="selectAllStudents()" class="bg-blue-600 hover:bg-blue-700 text-white px-3 py-2 rounded text-sm">
                                <i class="fas fa-check-square mr-1"></i>Pilih Semua
                            </button>
                            <button onclick="deselectAllStudents()" class="bg-gray-600 hover:bg-gray-700 text-white px-3 py-2 rounded text-sm">
                                <i class="fas fa-square mr-1"></i>Batal Pilih
                            </button>
                            <button onclick="deleteSelectedStudents()" class="bg-red-600 hover:bg-red-700 text-white px-3 py-2 rounded text-sm">
                                <i class="fas fa-trash mr-1"></i>Hapus Terpilih
                            </button>
                            <button onclick="deleteAllStudents()" class="bg-red-800 hover:bg-red-900 text-white px-3 py-2 rounded text-sm">
                                <i class="fas fa-trash-alt mr-1"></i>Hapus Semua
                            </button>
                        </div>
                    </div>
                </div>
                <div class="p-6">
                    <!-- Filter -->
                    <div class="mb-6 flex flex-col sm:flex-row gap-4">
                        <select id="studentsClassFilter" onchange="renderStudentsTable()" class="px-3 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500">
                            <option value="">Semua Kelas</option>
                        </select>
                        <input type="text" id="studentSearch" placeholder="Cari nama siswa..." onkeyup="renderStudentsTable()" class="px-3 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500">
                    </div>
                    
                    <div class="overflow-x-auto">
                        <table class="w-full">
                            <thead>
                                <tr class="border-b border-gray-200">
                                    <th class="text-left py-3 px-4 font-medium text-gray-700">
                                        <input type="checkbox" id="selectAllStudentsCheckbox" onchange="toggleAllStudents()" class="rounded">
                                    </th>
                                    <th class="text-left py-3 px-4 font-medium text-gray-700">No</th>
                                    <th class="text-left py-3 px-4 font-medium text-gray-700">Nama Siswa</th>
                                    <th class="text-left py-3 px-4 font-medium text-gray-700">Kelas</th>
                                    <th class="text-left py-3 px-4 font-medium text-gray-700">Jenis Kelamin</th>
                                    <th class="text-left py-3 px-4 font-medium text-gray-700">Nilai Akhir</th>
                                    <th class="text-left py-3 px-4 font-medium text-gray-700">Status</th>
                                    <th class="text-left py-3 px-4 font-medium text-gray-700">Aksi</th>
                                </tr>
                            </thead>
                            <tbody id="studentsTable">
                                <!-- Students will be populated here -->
                            </tbody>
                        </table>
                    </div>
                </div>
            </div>
        </div>

        <!-- Grades Tab -->
        <div id="grades" class="tab-content hidden">
            <div class="bg-white rounded-xl card-shadow">
                <div class="p-6 border-b border-gray-200">
                    <h3 class="text-xl font-bold text-gray-800">Penilaian Siswa</h3>
                    <p class="text-gray-600 mt-2">Kelola nilai berdasarkan 5 komponen penilaian</p>
                </div>
                <div class="p-6">
                    <!-- Student Selection -->
                    <div class="mb-6 grid grid-cols-1 md:grid-cols-3 gap-4">
                        <div>
                            <label class="block text-sm font-medium text-gray-700 mb-2">Pilih Mata Pelajaran</label>
                            <select id="subjectSelect" onchange="loadStudentGrades()" class="w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-purple-500">
                                <option value="">Pilih mata pelajaran...</option>
                            </select>
                        </div>
                        <div>
                            <label class="block text-sm font-medium text-gray-700 mb-2">Pilih Kelas</label>
                            <select id="gradesClassFilter" onchange="populateStudentSelect()" class="w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500">
                                <option value="">Semua Kelas</option>
                            </select>
                        </div>
                        <div>
                            <label class="block text-sm font-medium text-gray-700 mb-2">Pilih Siswa</label>
                            <select id="studentSelect" class="w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500" onchange="loadStudentGrades()">
                                <option value="">Pilih siswa...</option>
                            </select>
                        </div>
                    </div>

                    <div id="gradeForm" class="hidden">
                        <!-- Grade Inputs -->
                        <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6 mb-6">
                            <div class="bg-blue-50 p-4 rounded-lg">
                                <label class="block text-sm font-medium text-gray-700 mb-2">Kehadiran (30%)</label>
                                <input type="number" id="attendance" min="0" max="100" class="w-full px-3 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500" onchange="calculateFinalGrade(); autoSaveGrades();">
                                <p class="text-xs text-gray-500 mt-1">Nilai kehadiran siswa</p>
                            </div>
                            <div class="bg-green-50 p-4 rounded-lg">
                                <label class="block text-sm font-medium text-gray-700 mb-2">Tugas (20%)</label>
                                <input type="number" id="assignment" min="0" max="100" class="w-full px-3 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-green-500" onchange="calculateFinalGrade(); autoSaveGrades();">
                                <p class="text-xs text-gray-500 mt-1">Rata-rata nilai tugas</p>
                            </div>
                            <div class="bg-yellow-50 p-4 rounded-lg">
                                <label class="block text-sm font-medium text-gray-700 mb-2">Praktek (20%)</label>
                                <input type="number" id="practice" min="0" max="100" class="w-full px-3 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-yellow-500" onchange="calculateFinalGrade(); autoSaveGrades();">
                                <p class="text-xs text-gray-500 mt-1">Nilai praktik/keterampilan</p>
                            </div>
                            <div class="bg-purple-50 p-4 rounded-lg">
                                <label class="block text-sm font-medium text-gray-700 mb-2">UTS (15%)</label>
                                <input type="number" id="midterm" min="0" max="100" class="w-full px-3 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-purple-500" onchange="calculateFinalGrade(); autoSaveGrades();">
                                <p class="text-xs text-gray-500 mt-1">Ujian Tengah Semester</p>
                            </div>
                            <div class="bg-red-50 p-4 rounded-lg">
                                <label class="block text-sm font-medium text-gray-700 mb-2">UAS (15%)</label>
                                <input type="number" id="finalexam" min="0" max="100" class="w-full px-3 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-red-500" onchange="calculateFinalGrade(); autoSaveGrades();">
                                <p class="text-xs text-gray-500 mt-1">Ujian Akhir Semester</p>
                            </div>
                            <div class="bg-gray-100 p-4 rounded-lg">
                                <label class="block text-sm font-medium text-gray-700 mb-2">Nilai Akhir</label>
                                <div class="text-3xl font-bold text-gray-800" id="finalGrade">0</div>
                                <div class="text-sm text-gray-600" id="gradeStatus">-</div>
                                <div class="text-xs text-gray-500 mt-1">Hasil perhitungan otomatis</div>
                            </div>
                        </div>

                        <!-- Action Buttons -->
                        <div class="flex flex-wrap gap-3">
                            <button onclick="saveGrades()" class="bg-blue-600 hover:bg-blue-700 text-white px-6 py-2 rounded-lg transition-colors">
                                <i class="fas fa-save mr-2"></i>Simpan Nilai
                            </button>
                            <button onclick="generateRandomGrades()" class="bg-green-600 hover:bg-green-700 text-white px-6 py-2 rounded-lg transition-colors">
                                <i class="fas fa-random mr-2"></i>Generate Nilai Acak
                            </button>
                            <button onclick="clearGrades()" class="bg-gray-600 hover:bg-gray-700 text-white px-6 py-2 rounded-lg transition-colors">
                                <i class="fas fa-eraser mr-2"></i>Bersihkan Form
                            </button>
                        </div>
                    </div>
                </div>
            </div>
        </div>

        <!-- Reports Tab -->
        <div id="reports" class="tab-content hidden">
            <div class="bg-white rounded-xl card-shadow">
                <div class="p-6 border-b border-gray-200">
                    <h3 class="text-xl font-bold text-gray-800">Laporan Hasil Penilaian</h3>
                    <p class="text-gray-600 mt-2">Download dan cetak laporan dalam berbagai format</p>
                </div>
                <div class="p-6">
                    <!-- Filter Options -->
                    <div class="mb-6 grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-4">
                        <div>
                            <label class="block text-sm font-medium text-gray-700 mb-2">Filter Kelas</label>
                            <select id="reportsClassFilter" onchange="updateReportPreview()" class="w-full px-3 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500">
                                <option value="">Semua Kelas</option>
                            </select>
                        </div>
                        <div>
                            <label class="block text-sm font-medium text-gray-700 mb-2">Filter Mata Pelajaran</label>
                            <select id="reportsSubjectFilter" onchange="updateReportPreview()" class="w-full px-3 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500">
                                <option value="">Semua Mata Pelajaran</option>
                            </select>
                        </div>
                        <div>
                            <label class="block text-sm font-medium text-gray-700 mb-2">Periode Laporan</label>
                            <select id="reportsPeriodFilter" onchange="updateReportPreview()" class="w-full px-3 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500">
                                <option value="semester1">Semester 1</option>
                                <option value="semester2">Semester 2</option>
                                <option value="tahunan">Tahunan</option>
                            </select>
                        </div>
                        <div>
                            <label class="block text-sm font-medium text-gray-700 mb-2">Judul Laporan</label>
                            <input type="text" id="reportTitle" placeholder="Judul Laporan" value="Laporan Nilai Siswa" onchange="updateReportPreview()" class="w-full px-3 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500">
                        </div>
                    </div>
                    
                    <!-- Download Options -->
                    <div class="grid grid-cols-1 md:grid-cols-3 gap-6 mb-8">
                        <button onclick="downloadExcel()" class="bg-green-600 hover:bg-green-700 text-white p-6 rounded-xl transition-colors hover-scale">
                            <i class="fas fa-file-excel text-3xl mb-3"></i>
                            <div class="text-lg font-semibold">Download Excel</div>
                            <div class="text-sm opacity-90">Format .xlsx untuk analisis data</div>
                        </button>
                        <button onclick="downloadPDF()" class="bg-red-600 hover:bg-red-700 text-white p-6 rounded-xl transition-colors hover-scale">
                            <i class="fas fa-file-pdf text-3xl mb-3"></i>
                            <div class="text-lg font-semibold">Download PDF</div>
                            <div class="text-sm opacity-90">Format .pdf untuk cetak</div>
                        </button>
                        <button onclick="printReport()" class="bg-blue-600 hover:bg-blue-700 text-white p-6 rounded-xl transition-colors hover-scale">
                            <i class="fas fa-print text-3xl mb-3"></i>
                            <div class="text-lg font-semibold">Cetak Langsung</div>
                            <div class="text-sm opacity-90">Print ke printer</div>
                        </button>
                    </div>

                    <!-- Report Preview -->
                    <div class="border border-gray-200 rounded-lg">
                        <div class="p-4 bg-gray-50 border-b border-gray-200">
                            <h4 class="text-lg font-semibold text-gray-800">Preview Laporan</h4>
                            <p class="text-sm text-gray-600 mt-1">Preview otomatis berubah sesuai filter yang dipilih</p>
                        </div>
                        <div id="reportPreview" class="p-6">
                            <div class="text-center py-12">
                                <i class="fas fa-file-alt text-gray-400 text-4xl mb-4"></i>
                                <p class="text-gray-600">Memuat preview laporan...</p>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>

        <!-- Management Tab -->
        <div id="management" class="tab-content hidden">
            <div class="grid grid-cols-1 lg:grid-cols-3 gap-6">
                <!-- Teachers Management -->
                <div class="bg-white rounded-xl card-shadow">
                    <div class="p-6 border-b border-gray-200">
                        <div class="flex justify-between items-center mb-4">
                            <h3 class="text-lg font-bold text-gray-800">Data Guru</h3>
                            <button onclick="openAddTeacherModal()" class="bg-blue-600 hover:bg-blue-700 text-white px-3 py-2 rounded-lg text-sm">
                                <i class="fas fa-plus mr-1"></i>Tambah
                            </button>
                        </div>
                        
                        <!-- Teacher Bulk Actions -->
                        <div id="teacherBulkActions" class="hidden bg-blue-50 p-3 rounded-lg mb-3">
                            <div class="flex flex-wrap items-center gap-2">
                                <span class="text-sm font-medium text-blue-800">
                                    <span id="selectedTeachersCount">0</span> guru dipilih
                                </span>
                                <button onclick="selectAllTeachers()" class="bg-blue-600 hover:bg-blue-700 text-white px-2 py-1 rounded text-xs">
                                    <i class="fas fa-check-square mr-1"></i>Pilih Semua
                                </button>
                                <button onclick="deselectAllTeachers()" class="bg-gray-600 hover:bg-gray-700 text-white px-2 py-1 rounded text-xs">
                                    <i class="fas fa-square mr-1"></i>Batal Pilih
                                </button>
                                <button onclick="deleteSelectedTeachers()" class="bg-red-600 hover:bg-red-700 text-white px-2 py-1 rounded text-xs">
                                    <i class="fas fa-trash mr-1"></i>Hapus Terpilih
                                </button>
                                <button onclick="deleteAllTeachers()" class="bg-red-800 hover:bg-red-900 text-white px-2 py-1 rounded text-xs">
                                    <i class="fas fa-trash-alt mr-1"></i>Hapus Semua
                                </button>
                            </div>
                        </div>
                    </div>
                    <div class="p-6 max-h-96 overflow-y-auto">
                        <div id="teachersList" class="space-y-3">
                            <!-- Teachers will be populated here -->
                        </div>
                    </div>
                </div>

                <!-- Subjects Management -->
                <div class="bg-white rounded-xl card-shadow">
                    <div class="p-6 border-b border-gray-200">
                        <div class="flex justify-between items-center mb-4">
                            <h3 class="text-lg font-bold text-gray-800">Mata Pelajaran</h3>
                            <button onclick="openAddSubjectModal()" class="bg-green-600 hover:bg-green-700 text-white px-3 py-2 rounded-lg text-sm">
                                <i class="fas fa-plus mr-1"></i>Tambah
                            </button>
                        </div>
                        
                        <!-- Subject Bulk Actions -->
                        <div id="subjectBulkActions" class="hidden bg-green-50 p-3 rounded-lg mb-3">
                            <div class="flex flex-wrap items-center gap-2">
                                <span class="text-sm font-medium text-green-800">
                                    <span id="selectedSubjectsCount">0</span> mata pelajaran dipilih
                                </span>
                                <button onclick="selectAllSubjects()" class="bg-green-600 hover:bg-green-700 text-white px-2 py-1 rounded text-xs">
                                    <i class="fas fa-check-square mr-1"></i>Pilih Semua
                                </button>
                                <button onclick="deselectAllSubjects()" class="bg-gray-600 hover:bg-gray-700 text-white px-2 py-1 rounded text-xs">
                                    <i class="fas fa-square mr-1"></i>Batal Pilih
                                </button>
                                <button onclick="deleteSelectedSubjects()" class="bg-red-600 hover:bg-red-700 text-white px-2 py-1 rounded text-xs">
                                    <i class="fas fa-trash mr-1"></i>Hapus Terpilih
                                </button>
                                <button onclick="deleteAllSubjects()" class="bg-red-800 hover:bg-red-900 text-white px-2 py-1 rounded text-xs">
                                    <i class="fas fa-trash-alt mr-1"></i>Hapus Semua
                                </button>
                            </div>
                        </div>
                    </div>
                    <div class="p-6 max-h-96 overflow-y-auto">
                        <div id="subjectsList" class="space-y-3">
                            <!-- Subjects will be populated here -->
                        </div>
                    </div>
                </div>

                <!-- Classes Management -->
                <div class="bg-white rounded-xl card-shadow">
                    <div class="p-6 border-b border-gray-200">
                        <div class="flex justify-between items-center mb-4">
                            <h3 class="text-lg font-bold text-gray-800">Data Kelas</h3>
                            <button onclick="openAddClassModal()" class="bg-purple-600 hover:bg-purple-700 text-white px-3 py-2 rounded-lg text-sm">
                                <i class="fas fa-plus mr-1"></i>Tambah
                            </button>
                        </div>
                        
                        <!-- Class Bulk Actions -->
                        <div id="classBulkActions" class="hidden bg-purple-50 p-3 rounded-lg mb-3">
                            <div class="flex flex-wrap items-center gap-2">
                                <span class="text-sm font-medium text-purple-800">
                                    <span id="selectedClassesCount">0</span> kelas dipilih
                                </span>
                                <button onclick="selectAllClasses()" class="bg-purple-600 hover:bg-purple-700 text-white px-2 py-1 rounded text-xs">
                                    <i class="fas fa-check-square mr-1"></i>Pilih Semua
                                </button>
                                <button onclick="deselectAllClasses()" class="bg-gray-600 hover:bg-gray-700 text-white px-2 py-1 rounded text-xs">
                                    <i class="fas fa-square mr-1"></i>Batal Pilih
                                </button>
                                <button onclick="deleteSelectedClasses()" class="bg-red-600 hover:bg-red-700 text-white px-2 py-1 rounded text-xs">
                                    <i class="fas fa-trash mr-1"></i>Hapus Terpilih
                                </button>
                                <button onclick="deleteAllClasses()" class="bg-red-800 hover:bg-red-900 text-white px-2 py-1 rounded text-xs">
                                    <i class="fas fa-trash-alt mr-1"></i>Hapus Semua
                                </button>
                            </div>
                        </div>
                    </div>
                    <div class="p-6 max-h-96 overflow-y-auto">
                        <div id="classesList" class="space-y-3">
                            <!-- Classes will be populated here -->
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </main>

    <!-- Modals -->
    <!-- Add Student Modal -->
    <div id="addStudentModal" class="fixed inset-0 bg-black bg-opacity-50 hidden items-center justify-center z-50">
        <div class="bg-white rounded-xl p-6 w-full max-w-md mx-4">
            <div class="flex justify-between items-center mb-4">
                <h3 class="text-lg font-semibold text-gray-800">Tambah Siswa Baru</h3>
                <button onclick="closeAddStudentModal()" class="text-gray-400 hover:text-gray-600">
                    <i class="fas fa-times"></i>
                </button>
            </div>
            <form onsubmit="addStudent(event)">
                <div class="mb-4">
                    <label class="block text-sm font-medium text-gray-700 mb-2">Nama Siswa *</label>
                    <input type="text" id="newStudentName" required class="w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-transparent">
                </div>
                <div class="mb-4">
                    <label class="block text-sm font-medium text-gray-700 mb-2">Kelas *</label>
                    <select id="newStudentClass" required class="w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-transparent">
                        <option value="">Pilih kelas</option>
                    </select>
                </div>
                <div class="mb-6">
                    <label class="block text-sm font-medium text-gray-700 mb-2">Jenis Kelamin *</label>
                    <select id="newStudentGender" required class="w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-transparent">
                        <option value="">Pilih jenis kelamin</option>
                        <option value="Laki-laki">Laki-laki</option>
                        <option value="Perempuan">Perempuan</option>
                    </select>
                </div>
                <div class="flex space-x-3">
                    <button type="button" onclick="closeAddStudentModal()" class="flex-1 bg-gray-300 hover:bg-gray-400 text-gray-700 py-2 px-4 rounded-lg transition-colors">
                        Batal
                    </button>
                    <button type="submit" class="flex-1 bg-blue-600 hover:bg-blue-700 text-white py-2 px-4 rounded-lg transition-colors">
                        Tambah
                    </button>
                </div>
            </form>
        </div>
    </div>

    <!-- Edit Student Modal -->
    <div id="editStudentModal" class="fixed inset-0 bg-black bg-opacity-50 hidden items-center justify-center z-50">
        <div class="bg-white rounded-xl p-6 w-full max-w-md mx-4">
            <div class="flex justify-between items-center mb-4">
                <h3 class="text-lg font-semibold text-gray-800">Edit Data Siswa</h3>
                <button onclick="closeEditStudentModal()" class="text-gray-400 hover:text-gray-600">
                    <i class="fas fa-times"></i>
                </button>
            </div>
            <form onsubmit="updateStudent(event)">
                <input type="hidden" id="editStudentId">
                <div class="mb-4">
                    <label class="block text-sm font-medium text-gray-700 mb-2">Nama Siswa *</label>
                    <input type="text" id="editStudentName" required class="w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-transparent">
                </div>
                <div class="mb-4">
                    <label class="block text-sm font-medium text-gray-700 mb-2">Kelas *</label>
                    <select id="editStudentClass" required class="w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-transparent">
                        <option value="">Pilih kelas</option>
                    </select>
                </div>
                <div class="mb-6">
                    <label class="block text-sm font-medium text-gray-700 mb-2">Jenis Kelamin *</label>
                    <select id="editStudentGender" required class="w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-transparent">
                        <option value="">Pilih jenis kelamin</option>
                        <option value="Laki-laki">Laki-laki</option>
                        <option value="Perempuan">Perempuan</option>
                    </select>
                </div>
                <div class="flex space-x-3">
                    <button type="button" onclick="closeEditStudentModal()" class="flex-1 bg-gray-300 hover:bg-gray-400 text-gray-700 py-2 px-4 rounded-lg transition-colors">
                        Batal
                    </button>
                    <button type="submit" class="flex-1 bg-blue-600 hover:bg-blue-700 text-white py-2 px-4 rounded-lg transition-colors">
                        Update
                    </button>
                </div>
            </form>
        </div>
    </div>

    <!-- Add Teacher Modal -->
    <div id="addTeacherModal" class="fixed inset-0 bg-black bg-opacity-50 hidden items-center justify-center z-50">
        <div class="bg-white rounded-xl p-6 w-full max-w-md mx-4">
            <div class="flex justify-between items-center mb-4">
                <h3 class="text-lg font-semibold text-gray-800">Tambah Guru Baru</h3>
                <button onclick="closeAddTeacherModal()" class="text-gray-400 hover:text-gray-600">
                    <i class="fas fa-times"></i>
                </button>
            </div>
            <form onsubmit="addTeacher(event)">
                <div class="mb-4">
                    <label class="block text-sm font-medium text-gray-700 mb-2">Nama Guru *</label>
                    <input type="text" id="newTeacherName" required class="w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-transparent">
                </div>
                <div class="mb-6">
                    <label class="block text-sm font-medium text-gray-700 mb-2">NIP *</label>
                    <input type="text" id="newTeacherNIP" required class="w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-transparent">
                </div>
                <div class="flex space-x-3">
                    <button type="button" onclick="closeAddTeacherModal()" class="flex-1 bg-gray-300 hover:bg-gray-400 text-gray-700 py-2 px-4 rounded-lg transition-colors">
                        Batal
                    </button>
                    <button type="submit" class="flex-1 bg-blue-600 hover:bg-blue-700 text-white py-2 px-4 rounded-lg transition-colors">
                        Tambah
                    </button>
                </div>
            </form>
        </div>
    </div>

    <!-- Edit Teacher Modal -->
    <div id="editTeacherModal" class="fixed inset-0 bg-black bg-opacity-50 hidden items-center justify-center z-50">
        <div class="bg-white rounded-xl p-6 w-full max-w-md mx-4">
            <div class="flex justify-between items-center mb-4">
                <h3 class="text-lg font-semibold text-gray-800">Edit Data Guru</h3>
                <button onclick="closeEditTeacherModal()" class="text-gray-400 hover:text-gray-600">
                    <i class="fas fa-times"></i>
                </button>
            </div>
            <form onsubmit="updateTeacher(event)">
                <input type="hidden" id="editTeacherId">
                <div class="mb-4">
                    <label class="block text-sm font-medium text-gray-700 mb-2">Nama Guru *</label>
                    <input type="text" id="editTeacherName" required class="w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-transparent">
                </div>
                <div class="mb-6">
                    <label class="block text-sm font-medium text-gray-700 mb-2">NIP *</label>
                    <input type="text" id="editTeacherNIP" required class="w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-transparent">
                </div>
                <div class="flex space-x-3">
                    <button type="button" onclick="closeEditTeacherModal()" class="flex-1 bg-gray-300 hover:bg-gray-400 text-gray-700 py-2 px-4 rounded-lg transition-colors">
                        Batal
                    </button>
                    <button type="submit" class="flex-1 bg-blue-600 hover:bg-blue-700 text-white py-2 px-4 rounded-lg transition-colors">
                        Update
                    </button>
                </div>
            </form>
        </div>
    </div>

    <!-- Add Subject Modal -->
    <div id="addSubjectModal" class="fixed inset-0 bg-black bg-opacity-50 hidden items-center justify-center z-50">
        <div class="bg-white rounded-xl p-6 w-full max-w-md mx-4">
            <div class="flex justify-between items-center mb-4">
                <h3 class="text-lg font-semibold text-gray-800">Tambah Mata Pelajaran</h3>
                <button onclick="closeAddSubjectModal()" class="text-gray-400 hover:text-gray-600">
                    <i class="fas fa-times"></i>
                </button>
            </div>
            <form onsubmit="addSubject(event)">
                <div class="mb-4">
                    <label class="block text-sm font-medium text-gray-700 mb-2">Nama Mata Pelajaran *</label>
                    <input type="text" id="newSubjectName" required class="w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-transparent">
                </div>
                <div class="mb-6">
                    <label class="block text-sm font-medium text-gray-700 mb-2">Kode Mata Pelajaran *</label>
                    <input type="text" id="newSubjectCode" required class="w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-transparent">
                </div>
                <div class="flex space-x-3">
                    <button type="button" onclick="closeAddSubjectModal()" class="flex-1 bg-gray-300 hover:bg-gray-400 text-gray-700 py-2 px-4 rounded-lg transition-colors">
                        Batal
                    </button>
                    <button type="submit" class="flex-1 bg-green-600 hover:bg-green-700 text-white py-2 px-4 rounded-lg transition-colors">
                        Tambah
                    </button>
                </div>
            </form>
        </div>
    </div>

    <!-- Edit Subject Modal -->
    <div id="editSubjectModal" class="fixed inset-0 bg-black bg-opacity-50 hidden items-center justify-center z-50">
        <div class="bg-white rounded-xl p-6 w-full max-w-md mx-4">
            <div class="flex justify-between items-center mb-4">
                <h3 class="text-lg font-semibold text-gray-800">Edit Mata Pelajaran</h3>
                <button onclick="closeEditSubjectModal()" class="text-gray-400 hover:text-gray-600">
                    <i class="fas fa-times"></i>
                </button>
            </div>
            <form onsubmit="updateSubject(event)">
                <input type="hidden" id="editSubjectId">
                <div class="mb-4">
                    <label class="block text-sm font-medium text-gray-700 mb-2">Nama Mata Pelajaran *</label>
                    <input type="text" id="editSubjectName" required class="w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-transparent">
                </div>
                <div class="mb-6">
                    <label class="block text-sm font-medium text-gray-700 mb-2">Kode Mata Pelajaran *</label>
                    <input type="text" id="editSubjectCode" required class="w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-transparent">
                </div>
                <div class="flex space-x-3">
                    <button type="button" onclick="closeEditSubjectModal()" class="flex-1 bg-gray-300 hover:bg-gray-400 text-gray-700 py-2 px-4 rounded-lg transition-colors">
                        Batal
                    </button>
                    <button type="submit" class="flex-1 bg-green-600 hover:bg-green-700 text-white py-2 px-4 rounded-lg transition-colors">
                        Update
                    </button>
                </div>
            </form>
        </div>
    </div>

    <!-- Add Class Modal -->
    <div id="addClassModal" class="fixed inset-0 bg-black bg-opacity-50 hidden items-center justify-center z-50">
        <div class="bg-white rounded-xl p-6 w-full max-w-md mx-4">
            <div class="flex justify-between items-center mb-4">
                <h3 class="text-lg font-semibold text-gray-800">Tambah Kelas Baru</h3>
                <button onclick="closeAddClassModal()" class="text-gray-400 hover:text-gray-600">
                    <i class="fas fa-times"></i>
                </button>
            </div>
            <form onsubmit="addClass(event)">
                <div class="mb-4">
                    <label class="block text-sm font-medium text-gray-700 mb-2">Nama Kelas *</label>
                    <input type="text" id="newClassName" required class="w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-transparent">
                </div>
                <div class="mb-4">
                    <label class="block text-sm font-medium text-gray-700 mb-2">Wali Kelas</label>
                    <select id="newClassTeacher" class="w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-transparent">
                        <option value="">Pilih wali kelas</option>
                    </select>
                </div>
                <div class="mb-6">
                    <label class="block text-sm font-medium text-gray-700 mb-2">Kapasitas *</label>
                    <input type="number" id="newClassCapacity" value="30" min="1" max="50" required class="w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-transparent">
                </div>
                <div class="flex space-x-3">
                    <button type="button" onclick="closeAddClassModal()" class="flex-1 bg-gray-300 hover:bg-gray-400 text-gray-700 py-2 px-4 rounded-lg transition-colors">
                        Batal
                    </button>
                    <button type="submit" class="flex-1 bg-purple-600 hover:bg-purple-700 text-white py-2 px-4 rounded-lg transition-colors">
                        Tambah
                    </button>
                </div>
            </form>
        </div>
    </div>

    <!-- Edit Class Modal -->
    <div id="editClassModal" class="fixed inset-0 bg-black bg-opacity-50 hidden items-center justify-center z-50">
        <div class="bg-white rounded-xl p-6 w-full max-w-md mx-4">
            <div class="flex justify-between items-center mb-4">
                <h3 class="text-lg font-semibold text-gray-800">Edit Data Kelas</h3>
                <button onclick="closeEditClassModal()" class="text-gray-400 hover:text-gray-600">
                    <i class="fas fa-times"></i>
                </button>
            </div>
            <form onsubmit="updateClass(event)">
                <input type="hidden" id="editClassId">
                <div class="mb-4">
                    <label class="block text-sm font-medium text-gray-700 mb-2">Nama Kelas *</label>
                    <input type="text" id="editClassName" required class="w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-transparent">
                </div>
                <div class="mb-4">
                    <label class="block text-sm font-medium text-gray-700 mb-2">Wali Kelas</label>
                    <select id="editClassTeacher" class="w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-transparent">
                        <option value="">Pilih wali kelas</option>
                    </select>
                </div>
                <div class="mb-6">
                    <label class="block text-sm font-medium text-gray-700 mb-2">Kapasitas *</label>
                    <input type="number" id="editClassCapacity" min="1" max="50" required class="w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-transparent">
                </div>
                <div class="flex space-x-3">
                    <button type="button" onclick="closeEditClassModal()" class="flex-1 bg-gray-300 hover:bg-gray-400 text-gray-700 py-2 px-4 rounded-lg transition-colors">
                        Batal
                    </button>
                    <button type="submit" class="flex-1 bg-purple-600 hover:bg-purple-700 text-white py-2 px-4 rounded-lg transition-colors">
                        Update
                    </button>
                </div>
            </form>
        </div>
    </div>

    <!-- Settings Modal -->
    <div id="settingsModal" class="fixed inset-0 bg-black bg-opacity-50 hidden items-center justify-center z-50">
        <div class="bg-white rounded-xl p-6 w-full max-w-md mx-4">
            <div class="flex justify-between items-center mb-4">
                <h3 class="text-lg font-semibold text-gray-800">Pengaturan Sekolah</h3>
                <button onclick="closeSettings()" class="text-gray-400 hover:text-gray-600">
                    <i class="fas fa-times"></i>
                </button>
            </div>
            <div class="mb-4">
                <label class="block text-sm font-medium text-gray-700 mb-2">Nama Sekolah</label>
                <input type="text" id="schoolNameInput" class="w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-transparent">
            </div>
            <div class="mb-6">
                <label class="block text-sm font-medium text-gray-700 mb-2">Alamat Sekolah</label>
                <textarea id="schoolAddressInput" rows="3" class="w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-transparent"></textarea>
            </div>
            <div class="flex space-x-3">
                <button onclick="closeSettings()" class="flex-1 bg-gray-300 hover:bg-gray-400 text-gray-700 py-2 px-4 rounded-lg transition-colors">
                    Batal
                </button>
                <button onclick="saveSettings()" class="flex-1 bg-blue-600 hover:bg-blue-700 text-white py-2 px-4 rounded-lg transition-colors">
                    Simpan
                </button>
            </div>
        </div>
    </div>

    <script>
        // Global Variables
        let students = [];
        let grades = {}; // Structure: grades[studentId][subjectId] = {attendance, assignment, practice, midterm, finalexam}
        let teachers = [];
        let subjects = [];
        let classes = [];
        let schoolSettings = {
            name: 'SMA Negeri 1 Jakarta',
            address: 'Jl. Pendidikan No. 123, Jakarta Pusat'
        };
        
        // Selection tracking
        let selectedStudents = new Set();
        let selectedTeachers = new Set();
        let selectedSubjects = new Set();
        let selectedClasses = new Set();

        // Auto-save variables
        let autoSaveInterval;
        let lastSaveTime = null;

        // Utility Functions
        function showNotification(message, type = 'success') {
            const notification = document.getElementById('notification');
            notification.textContent = message;
            notification.className = `notification ${type}`;
            notification.classList.add('show');
            
            setTimeout(() => {
                notification.classList.remove('show');
            }, 3000);
        }

        function showAutoSaveIndicator() {
            const indicator = document.getElementById('autoSaveIndicator');
            indicator.classList.add('show');
            setTimeout(() => {
                indicator.classList.remove('show');
            }, 2000);
        }

        function updateLastSaveTime() {
            lastSaveTime = new Date();
            const timeElement = document.getElementById('lastSavedTime');
            if (timeElement) {
                timeElement.textContent = lastSaveTime.toLocaleTimeString('id-ID');
            }
        }

        function saveData() {
            try {
                localStorage.setItem('studentsData', JSON.stringify(students));
                localStorage.setItem('gradesData', JSON.stringify(grades));
                localStorage.setItem('teachersData', JSON.stringify(teachers));
                localStorage.setItem('subjectsData', JSON.stringify(subjects));
                localStorage.setItem('classesData', JSON.stringify(classes));
                localStorage.setItem('schoolSettings', JSON.stringify(schoolSettings));
                localStorage.setItem('lastSaved', new Date().toISOString());
                updateLastSaveTime();
                console.log('Data saved successfully at:', new Date().toLocaleTimeString());
                return true;
            } catch (error) {
                console.error('Error saving data:', error);
                showNotification('Error menyimpan data: ' + error.message, 'error');
                return false;
            }
        }

        // Auto-save function that runs periodically
        function autoSave() {
            if (saveData()) {
                showAutoSaveIndicator();
                console.log('Auto-save completed at:', new Date().toLocaleTimeString());
            }
        }

        // Auto-save grades when input changes
        function autoSaveGrades() {
            const studentId = parseInt(document.getElementById('studentSelect').value);
            const subjectId = parseInt(document.getElementById('subjectSelect').value);
            if (!studentId || !subjectId) return;
            
            // Debounce auto-save to avoid too frequent saves
            clearTimeout(window.gradesSaveTimeout);
            window.gradesSaveTimeout = setTimeout(() => {
                if (!grades[studentId]) grades[studentId] = {};
                
                grades[studentId][subjectId] = {
                    attendance: parseFloat(document.getElementById('attendance').value) || 0,
                    assignment: parseFloat(document.getElementById('assignment').value) || 0,
                    practice: parseFloat(document.getElementById('practice').value) || 0,
                    midterm: parseFloat(document.getElementById('midterm').value) || 0,
                    finalexam: parseFloat(document.getElementById('finalexam').value) || 0
                };
                
                if (saveData()) {
                    showAutoSaveIndicator();
                    renderStudentsTable();
                    updateDashboard();
                }
            }, 1000); // Save after 1 second of no changes
        }

        // Save data whenever there are changes
        function saveDataWithNotification(message = 'Data berhasil disimpan!') {
            if (saveData()) {
                if (message) {
                    showNotification(message);
                }
            }
        }

        function loadData() {
            try {
                const savedStudents = localStorage.getItem('studentsData');
                const savedGrades = localStorage.getItem('gradesData');
                const savedTeachers = localStorage.getItem('teachersData');
                const savedSubjects = localStorage.getItem('subjectsData');
                const savedClasses = localStorage.getItem('classesData');
                const savedSettings = localStorage.getItem('schoolSettings');
                const savedTime = localStorage.getItem('lastSaved');

                if (savedStudents) students = JSON.parse(savedStudents);
                if (savedGrades) grades = JSON.parse(savedGrades);
                if (savedTeachers) teachers = JSON.parse(savedTeachers);
                if (savedSubjects) subjects = JSON.parse(savedSubjects);
                if (savedClasses) classes = JSON.parse(savedClasses);
                if (savedSettings) schoolSettings = JSON.parse(savedSettings);
                if (savedTime) {
                    lastSaveTime = new Date(savedTime);
                    updateLastSaveTime();
                }
                
                console.log('Data loaded successfully');
                
                // If no data exists, load sample data
                if (students.length === 0 && teachers.length === 0 && classes.length === 0) {
                    loadSampleData();
                    saveData(); // Save the sample data
                }
            } catch (error) {
                console.error('Error loading data:', error);
                showNotification('Error memuat data, memuat data contoh', 'warning');
                loadSampleData();
                saveData();
            }
        }

        function loadSampleData() {
            // Sample teachers
            teachers = [
                { id: 1, name: 'Budi Santoso, S.Pd', nip: '196501011990031001' },
                { id: 2, name: 'Siti Aminah, S.Si', nip: '197203151995122002' },
                { id: 3, name: 'Ahmad Fauzi, S.Pd', nip: '198012102005011003' },
                { id: 4, name: 'Dewi Lestari, S.Pd', nip: '198506252010012004' }
            ];

            // Sample subjects
            subjects = [
                { id: 1, name: 'Matematika', code: 'MTK' },
                { id: 2, name: 'Bahasa Indonesia', code: 'BIN' },
                { id: 3, name: 'Bahasa Inggris', code: 'BIG' },
                { id: 4, name: 'Fisika', code: 'FIS' },
                { id: 5, name: 'Kimia', code: 'KIM' }
            ];

            // Sample classes
            classes = [
                { id: 1, name: 'X IPA 1', teacher: 1, capacity: 30 },
                { id: 2, name: 'X IPA 2', teacher: 2, capacity: 30 },
                { id: 3, name: 'XI IPA 1', teacher: 3, capacity: 28 },
                { id: 4, name: 'XI IPA 2', teacher: 4, capacity: 28 },
                { id: 5, name: 'XII IPA 1', teacher: 1, capacity: 25 }
            ];

            // Sample students
            const sampleNames = [
                'Ahmad Rizki', 'Siti Nurhaliza', 'Budi Santoso', 'Dewi Sartika', 'Andi Wijaya',
                'Maya Sari', 'Rudi Hartono', 'Indah Permata', 'Fajar Nugraha', 'Rina Wati',
                'Doni Pratama', 'Lisa Maharani', 'Eko Saputra', 'Fitri Handayani', 'Gilang Ramadhan'
            ];

            students = sampleNames.map((name, index) => ({
                id: index + 1,
                name: name,
                gender: index % 2 === 0 ? 'Laki-laki' : 'Perempuan',
                class: (index % 5) + 1
            }));

            // Generate sample grades for each student and subject
            students.forEach(student => {
                grades[student.id] = {};
                subjects.forEach(subject => {
                    grades[student.id][subject.id] = {
                        attendance: Math.floor(Math.random() * 21) + 80,
                        assignment: Math.floor(Math.random() * 21) + 75,
                        practice: Math.floor(Math.random() * 21) + 70,
                        midterm: Math.floor(Math.random() * 21) + 75,
                        finalexam: Math.floor(Math.random() * 21) + 70
                    };
                });
            });
            
            console.log('Sample data loaded');
        }

        // Initialize auto-save
        function initializeAutoSave() {
            // Auto-save every 30 seconds
            autoSaveInterval = setInterval(autoSave, 30000);
            
            // Save when page is about to unload
            window.addEventListener('beforeunload', function(e) {
                saveData();
            });
            
            // Save when page becomes hidden (mobile/tab switching)
            document.addEventListener('visibilitychange', function() {
                if (document.hidden) {
                    saveData();
                }
            });
            
            console.log('Auto-save initialized - saving every 30 seconds');
        }

        // Tab Management
        function showTab(tabName) {
            // Hide all tabs
            document.querySelectorAll('.tab-content').forEach(tab => {
                tab.classList.add('hidden');
            });
            
            // Show selected tab
            document.getElementById(tabName).classList.remove('hidden');
            
            // Update navigation
            document.querySelectorAll('.nav-btn').forEach(btn => {
                btn.classList.remove('border-blue-500', 'text-blue-600');
                btn.classList.add('border-transparent', 'text-gray-500');
            });
            
            event.target.classList.remove('border-transparent', 'text-gray-500');
            event.target.classList.add('border-blue-500', 'text-blue-600');

            // Refresh data for specific tabs
            if (tabName === 'dashboard') {
                updateDashboard();
            } else if (tabName === 'students') {
                renderStudentsTable();
            } else if (tabName === 'grades') {
                populateSubjectSelect();
                populateStudentSelect();
            } else if (tabName === 'reports') {
                populateClassSelects();
                populateReportsSubjectSelect();
                updateReportPreview();
            } else if (tabName === 'management') {
                renderTeachers();
                renderSubjects();
                renderClasses();
            }
        }

        // Dashboard Functions
        function updateDashboard() {
            // Update statistics
            document.getElementById('totalStudents').textContent = students.length;
            document.getElementById('totalClasses').textContent = classes.length;
            document.getElementById('totalTeachers').textContent = teachers.length;
            document.getElementById('totalSubjects').textContent = subjects.length;
            
            if (students.length > 0) {
                const allGrades = students.map(student => calculateStudentFinalGrade(student.id));
                const average = allGrades.reduce((sum, grade) => sum + grade, 0) / allGrades.length;
                const highest = Math.max(...allGrades);
                const needAttention = allGrades.filter(grade => grade < 70).length;
                
                document.getElementById('classAverage').textContent = average.toFixed(1);
                document.getElementById('highestGrade').textContent = highest.toFixed(1);
                document.getElementById('needAttention').textContent = needAttention;

                // Grade distribution
                const excellent = allGrades.filter(g => g >= 90).length;
                const good = allGrades.filter(g => g >= 80 && g < 90).length;
                const fair = allGrades.filter(g => g >= 70 && g < 80).length;
                const poor = allGrades.filter(g => g < 70).length;

                document.getElementById('excellentCount').textContent = excellent;
                document.getElementById('goodCount').textContent = good;
                document.getElementById('fairCount').textContent = fair;
                document.getElementById('poorCount').textContent = poor;
            }
            
            updateTopStudents();
        }

        function updateTopStudents() {
            const studentsWithGrades = students.map(student => ({
                ...student,
                finalGrade: calculateStudentFinalGrade(student.id)
            })).sort((a, b) => b.finalGrade - a.finalGrade).slice(0, 10);
            
            const container = document.getElementById('topStudentsList');
            container.innerHTML = '';
            
            if (studentsWithGrades.length === 0) {
                container.innerHTML = '<p class="text-gray-500 text-center py-4">Belum ada data siswa</p>';
                return;
            }
            
            studentsWithGrades.forEach((student, index) => {
                const className = classes.find(c => c.id === student.class)?.name || 'Tidak ada kelas';
                const div = document.createElement('div');
                div.className = 'flex items-center justify-between p-3 bg-gray-50 rounded-lg hover:bg-gray-100 transition-colors';
                
                let medalIcon = '';
                if (index === 0) medalIcon = '🥇';
                else if (index === 1) medalIcon = '🥈';
                else if (index === 2) medalIcon = '🥉';
                else medalIcon = `${index + 1}.`;
                
                div.innerHTML = `
                    <div class="flex items-center space-x-3">
                        <span class="text-lg font-bold w-8">${medalIcon}</span>
                        <div>
                            <div class="font-medium text-gray-800">${student.name}</div>
                            <div class="text-sm text-gray-500">${className}</div>
                        </div>
                    </div>
                    <div class="text-right">
                        <div class="font-bold text-lg ${student.finalGrade >= 90 ? 'text-green-600' : student.finalGrade >= 80 ? 'text-blue-600' : 'text-yellow-600'}">${student.finalGrade.toFixed(1)}</div>
                        <div class="text-xs text-gray-500">${getGradeStatus(student.finalGrade)}</div>
                    </div>
                `;
                container.appendChild(div);
            });
        }

        // Student Management
        function renderStudentsTable() {
            const selectedClass = document.getElementById('studentsClassFilter')?.value;
            const searchTerm = document.getElementById('studentSearch')?.value.toLowerCase() || '';
            
            let filteredStudents = students;
            
            if (selectedClass) {
                filteredStudents = filteredStudents.filter(s => s.class == selectedClass);
            }
            
            if (searchTerm) {
                filteredStudents = filteredStudents.filter(s => 
                    s.name.toLowerCase().includes(searchTerm)
                );
            }
            
            const tbody = document.getElementById('studentsTable');
            tbody.innerHTML = '';
            
            if (filteredStudents.length === 0) {
                tbody.innerHTML = '<tr><td colspan="8" class="text-center py-8 text-gray-500">Tidak ada data siswa</td></tr>';
                return;
            }
            
            filteredStudents.forEach((student, index) => {
                const finalGrade = calculateStudentFinalGrade(student.id);
                const className = classes.find(c => c.id === student.class)?.name || 'Belum ada kelas';
                const status = getGradeStatus(finalGrade);
                
                const row = document.createElement('tr');
                row.className = 'border-b border-gray-100 hover:bg-gray-50';
                row.innerHTML = `
                    <td class="py-3 px-4">
                        <input type="checkbox" class="student-checkbox rounded" value="${student.id}" onchange="updateStudentSelection()" ${selectedStudents.has(student.id) ? 'checked' : ''}>
                    </td>
                    <td class="py-3 px-4">${index + 1}</td>
                    <td class="py-3 px-4 font-medium">${student.name}</td>
                    <td class="py-3 px-4">${className}</td>
                    <td class="py-3 px-4">${student.gender}</td>
                    <td class="py-3 px-4">
                        <span class="px-2 py-1 rounded-full text-sm font-medium ${getGradeColor(finalGrade)}">
                            ${finalGrade.toFixed(1)}
                        </span>
                    </td>
                    <td class="py-3 px-4">
                        <span class="px-2 py-1 rounded-full text-xs ${getStatusColor(finalGrade)}">
                            ${status}
                        </span>
                    </td>
                    <td class="py-3 px-4">
                        <button onclick="editStudent(${student.id})" class="text-blue-600 hover:text-blue-800 mr-3" title="Edit">
                            <i class="fas fa-edit"></i>
                        </button>
                        <button onclick="deleteStudent(${student.id})" class="text-red-600 hover:text-red-800" title="Hapus">
                            <i class="fas fa-trash"></i>
                        </button>
                    </td>
                `;
                tbody.appendChild(row);
            });
            
            updateStudentSelection();
        }

        function openAddStudentModal() {
            populateClassSelects();
            document.getElementById('addStudentModal').classList.remove('hidden');
            document.getElementById('addStudentModal').classList.add('flex');
        }

        function closeAddStudentModal() {
            document.getElementById('addStudentModal').classList.add('hidden');
            document.getElementById('addStudentModal').classList.remove('flex');
            document.getElementById('newStudentName').value = '';
            document.getElementById('newStudentClass').value = '';
            document.getElementById('newStudentGender').value = '';
        }

        function addStudent(event) {
            event.preventDefault();
            const name = document.getElementById('newStudentName').value.trim();
            const classId = parseInt(document.getElementById('newStudentClass').value);
            const gender = document.getElementById('newStudentGender').value;
            
            if (!name || !classId || !gender) {
                showNotification('Semua field harus diisi', 'error');
                return;
            }
            
            const newStudent = {
                id: Date.now(),
                name: name,
                class: classId,
                gender: gender
            };
            
            students.push(newStudent);
            grades[newStudent.id] = {};
            // Initialize grades for all subjects
            subjects.forEach(subject => {
                grades[newStudent.id][subject.id] = {
                    attendance: 0,
                    assignment: 0,
                    practice: 0,
                    midterm: 0,
                    finalexam: 0
                };
            });
            
            saveDataWithNotification('Siswa berhasil ditambahkan!');
            renderStudentsTable();
            populateStudentSelect();
            updateDashboard();
            closeAddStudentModal();
        }

        function editStudent(id) {
            const student = students.find(s => s.id === id);
            if (!student) return;
            
            // Populate edit form
            document.getElementById('editStudentId').value = student.id;
            document.getElementById('editStudentName').value = student.name;
            document.getElementById('editStudentClass').value = student.class || '';
            document.getElementById('editStudentGender').value = student.gender;
            
            // Populate class options
            populateEditClassSelects();
            
            // Show modal
            document.getElementById('editStudentModal').classList.remove('hidden');
            document.getElementById('editStudentModal').classList.add('flex');
        }

        function closeEditStudentModal() {
            document.getElementById('editStudentModal').classList.add('hidden');
            document.getElementById('editStudentModal').classList.remove('flex');
        }

        function updateStudent(event) {
            event.preventDefault();
            const id = parseInt(document.getElementById('editStudentId').value);
            const name = document.getElementById('editStudentName').value.trim();
            const classId = parseInt(document.getElementById('editStudentClass').value) || null;
            const gender = document.getElementById('editStudentGender').value;
            
            if (!name || !gender) {
                showNotification('Nama dan jenis kelamin harus diisi', 'error');
                return;
            }
            
            const studentIndex = students.findIndex(s => s.id === id);
            if (studentIndex !== -1) {
                students[studentIndex] = {
                    ...students[studentIndex],
                    name: name,
                    class: classId,
                    gender: gender
                };
                
                saveDataWithNotification('Data siswa berhasil diperbarui!');
                renderStudentsTable();
                populateStudentSelect();
                updateDashboard();
                closeEditStudentModal();
            }
        }

        function deleteStudent(id) {
            const student = students.find(s => s.id === id);
            if (confirm(`Apakah Anda yakin ingin menghapus siswa "${student.name}"?`)) {
                students = students.filter(s => s.id !== id);
                delete grades[id];
                saveDataWithNotification('Siswa berhasil dihapus!');
                renderStudentsTable();
                populateStudentSelect();
                updateDashboard();
            }
        }

        // Grade Management
        function populateStudentSelect() {
            const selectedClass = document.getElementById('gradesClassFilter')?.value;
            let filteredStudents = students;
            
            if (selectedClass) {
                filteredStudents = students.filter(s => s.class == selectedClass);
            }
            
            const select = document.getElementById('studentSelect');
            select.innerHTML = '<option value="">Pilih siswa...</option>';
            
            filteredStudents.forEach(student => {
                const className = classes.find(c => c.id === student.class)?.name || '';
                const option = document.createElement('option');
                option.value = student.id;
                option.textContent = `${student.name} (${className})`;
                select.appendChild(option);
            });
        }

        function loadStudentGrades() {
            const studentId = parseInt(document.getElementById('studentSelect').value);
            const subjectId = parseInt(document.getElementById('subjectSelect').value);
            
            if (!studentId || !subjectId) {
                document.getElementById('gradeForm').classList.add('hidden');
                return;
            }
            
            document.getElementById('gradeForm').classList.remove('hidden');
            
            const studentGrades = (grades[studentId] && grades[studentId][subjectId]) || {};
            document.getElementById('attendance').value = studentGrades.attendance || 0;
            document.getElementById('assignment').value = studentGrades.assignment || 0;
            document.getElementById('practice').value = studentGrades.practice || 0;
            document.getElementById('midterm').value = studentGrades.midterm || 0;
            document.getElementById('finalexam').value = studentGrades.finalexam || 0;
            
            calculateFinalGrade();
        }

        function calculateFinalGrade() {
            const attendance = parseFloat(document.getElementById('attendance').value) || 0;
            const assignment = parseFloat(document.getElementById('assignment').value) || 0;
            const practice = parseFloat(document.getElementById('practice').value) || 0;
            const midterm = parseFloat(document.getElementById('midterm').value) || 0;
            const finalexam = parseFloat(document.getElementById('finalexam').value) || 0;
            
            const finalGrade = (attendance * 0.3) + (assignment * 0.2) + (practice * 0.2) + (midterm * 0.15) + (finalexam * 0.15);
            
            document.getElementById('finalGrade').textContent = finalGrade.toFixed(1);
            document.getElementById('gradeStatus').textContent = getGradeStatus(finalGrade);
        }

        function calculateStudentFinalGrade(studentId, subjectId = null) {
            if (!grades[studentId]) return 0;
            
            if (subjectId) {
                // Calculate for specific subject
                const studentGrades = grades[studentId][subjectId] || {};
                const attendance = studentGrades.attendance || 0;
                const assignment = studentGrades.assignment || 0;
                const practice = studentGrades.practice || 0;
                const midterm = studentGrades.midterm || 0;
                const finalexam = studentGrades.finalexam || 0;
                
                return (attendance * 0.3) + (assignment * 0.2) + (practice * 0.2) + (midterm * 0.15) + (finalexam * 0.15);
            } else {
                // Calculate average across all subjects
                const studentSubjects = grades[studentId];
                const subjectIds = Object.keys(studentSubjects);
                
                if (subjectIds.length === 0) return 0;
                
                const totalGrade = subjectIds.reduce((sum, subId) => {
                    return sum + calculateStudentFinalGrade(studentId, parseInt(subId));
                }, 0);
                
                return totalGrade / subjectIds.length;
            }
        }

        function saveGrades() {
            const studentId = parseInt(document.getElementById('studentSelect').value);
            const subjectId = parseInt(document.getElementById('subjectSelect').value);
            
            if (!studentId) {
                showNotification('Pilih siswa terlebih dahulu', 'error');
                return;
            }
            
            if (!subjectId) {
                showNotification('Pilih mata pelajaran terlebih dahulu', 'error');
                return;
            }
            
            if (!grades[studentId]) grades[studentId] = {};
            
            grades[studentId][subjectId] = {
                attendance: parseFloat(document.getElementById('attendance').value) || 0,
                assignment: parseFloat(document.getElementById('assignment').value) || 0,
                practice: parseFloat(document.getElementById('practice').value) || 0,
                midterm: parseFloat(document.getElementById('midterm').value) || 0,
                finalexam: parseFloat(document.getElementById('finalexam').value) || 0
            };
            
            const subjectName = subjects.find(s => s.id === subjectId)?.name || 'mata pelajaran';
            saveDataWithNotification(`Nilai ${subjectName} berhasil disimpan!`);
            renderStudentsTable();
            updateDashboard();
        }

        function generateRandomGrades() {
            document.getElementById('attendance').value = Math.floor(Math.random() * 21) + 80;
            document.getElementById('assignment').value = Math.floor(Math.random() * 21) + 75;
            document.getElementById('practice').value = Math.floor(Math.random() * 21) + 70;
            document.getElementById('midterm').value = Math.floor(Math.random() * 21) + 75;
            document.getElementById('finalexam').value = Math.floor(Math.random() * 21) + 70;
            
            calculateFinalGrade();
            autoSaveGrades();
            showNotification('Nilai acak berhasil digenerate!');
        }

        function clearGrades() {
            document.getElementById('attendance').value = '';
            document.getElementById('assignment').value = '';
            document.getElementById('practice').value = '';
            document.getElementById('midterm').value = '';
            document.getElementById('finalexam').value = '';
            document.getElementById('finalGrade').textContent = '0';
            document.getElementById('gradeStatus').textContent = '-';
        }

        // Utility Functions for Grades
        function getGradeStatus(grade) {
            if (grade >= 90) return 'Sangat Baik';
            else if (grade >= 80) return 'Baik';
            else if (grade >= 70) return 'Cukup';
            else return 'Perlu Perbaikan';
        }

        function getGradeColor(grade) {
            if (grade >= 90) return 'bg-green-100 text-green-800';
            else if (grade >= 80) return 'bg-blue-100 text-blue-800';
            else if (grade >= 70) return 'bg-yellow-100 text-yellow-800';
            else return 'bg-red-100 text-red-800';
        }

        function getStatusColor(grade) {
            if (grade >= 90) return 'bg-green-100 text-green-800';
            else if (grade >= 80) return 'bg-blue-100 text-blue-800';
            else if (grade >= 70) return 'bg-yellow-100 text-yellow-800';
            else return 'bg-red-100 text-red-800';
        }

        // Selection Functions
        function updateStudentSelection() {
            const checkboxes = document.querySelectorAll('.student-checkbox');
            selectedStudents.clear();
            
            checkboxes.forEach(checkbox => {
                if (checkbox.checked) {
                    selectedStudents.add(parseInt(checkbox.value));
                }
            });
            
            const count = selectedStudents.size;
            document.getElementById('selectedStudentsCount').textContent = count;
            
            const bulkActions = document.getElementById('studentBulkActions');
            if (count > 0) {
                bulkActions.classList.remove('hidden');
            } else {
                bulkActions.classList.add('hidden');
            }
            
            // Update select all checkbox
            const selectAllCheckbox = document.getElementById('selectAllStudentsCheckbox');
            if (selectAllCheckbox) {
                selectAllCheckbox.checked = count > 0 && count === checkboxes.length;
            }
        }

        function toggleAllStudents() {
            const selectAllCheckbox = document.getElementById('selectAllStudentsCheckbox');
            const checkboxes = document.querySelectorAll('.student-checkbox');
            
            checkboxes.forEach(checkbox => {
                checkbox.checked = selectAllCheckbox.checked;
            });
            
            updateStudentSelection();
        }

        function selectAllStudents() {
            const checkboxes = document.querySelectorAll('.student-checkbox');
            checkboxes.forEach(checkbox => {
                checkbox.checked = true;
            });
            updateStudentSelection();
        }

        function deselectAllStudents() {
            const checkboxes = document.querySelectorAll('.student-checkbox');
            checkboxes.forEach(checkbox => {
                checkbox.checked = false;
            });
            updateStudentSelection();
        }

        function deleteSelectedStudents() {
            if (selectedStudents.size === 0) return;
            
            if (confirm(`Apakah Anda yakin ingin menghapus ${selectedStudents.size} siswa yang dipilih?`)) {
                selectedStudents.forEach(id => {
                    students = students.filter(s => s.id !== id);
                    delete grades[id];
                });
                
                selectedStudents.clear();
                saveDataWithNotification(`Siswa yang dipilih berhasil dihapus!`);
                renderStudentsTable();
                populateStudentSelect();
                updateDashboard();
            }
        }

        function deleteAllStudents() {
            if (students.length === 0) return;
            
            if (confirm(`Apakah Anda yakin ingin menghapus SEMUA ${students.length} siswa? Tindakan ini tidak dapat dibatalkan!`)) {
                students = [];
                grades = {};
                selectedStudents.clear();
                saveDataWithNotification('Semua data siswa berhasil dihapus!');
                renderStudentsTable();
                populateStudentSelect();
                updateDashboard();
            }
        }

        // Teacher Management Functions
        function renderTeachers() {
            const container = document.getElementById('teachersList');
            container.innerHTML = '';
            
            if (teachers.length === 0) {
                container.innerHTML = '<p class="text-gray-500 text-center py-4">Belum ada data guru</p>';
                return;
            }
            
            teachers.forEach(teacher => {
                const div = document.createElement('div');
                div.className = 'flex items-center justify-between p-3 bg-gray-50 rounded-lg hover:bg-gray-100 transition-colors';
                div.innerHTML = `
                    <div class="flex items-center space-x-3">
                        <input type="checkbox" class="teacher-checkbox rounded" value="${teacher.id}" onchange="updateTeacherSelection()" ${selectedTeachers.has(teacher.id) ? 'checked' : ''}>
                        <div>
                            <div class="font-medium text-gray-800">${teacher.name}</div>
                            <div class="text-sm text-gray-500">NIP: ${teacher.nip}</div>
                        </div>
                    </div>
                    <div class="flex space-x-2">
                        <button onclick="editTeacher(${teacher.id})" class="text-blue-600 hover:text-blue-800" title="Edit">
                            <i class="fas fa-edit"></i>
                        </button>
                        <button onclick="deleteTeacher(${teacher.id})" class="text-red-600 hover:text-red-800" title="Hapus">
                            <i class="fas fa-trash"></i>
                        </button>
                    </div>
                `;
                container.appendChild(div);
            });
            
            updateTeacherSelection();
        }

        function updateTeacherSelection() {
            const checkboxes = document.querySelectorAll('.teacher-checkbox');
            selectedTeachers.clear();
            
            checkboxes.forEach(checkbox => {
                if (checkbox.checked) {
                    selectedTeachers.add(parseInt(checkbox.value));
                }
            });
            
            const count = selectedTeachers.size;
            document.getElementById('selectedTeachersCount').textContent = count;
            
            const bulkActions = document.getElementById('teacherBulkActions');
            if (count > 0) {
                bulkActions.classList.remove('hidden');
            } else {
                bulkActions.classList.add('hidden');
            }
        }

        function selectAllTeachers() {
            const checkboxes = document.querySelectorAll('.teacher-checkbox');
            checkboxes.forEach(checkbox => {
                checkbox.checked = true;
            });
            updateTeacherSelection();
        }

        function deselectAllTeachers() {
            const checkboxes = document.querySelectorAll('.teacher-checkbox');
            checkboxes.forEach(checkbox => {
                checkbox.checked = false;
            });
            updateTeacherSelection();
        }

        function deleteSelectedTeachers() {
            if (selectedTeachers.size === 0) return;
            
            if (confirm(`Apakah Anda yakin ingin menghapus ${selectedTeachers.size} guru yang dipilih?`)) {
                selectedTeachers.forEach(id => {
                    teachers = teachers.filter(t => t.id !== id);
                    // Remove teacher assignment from classes
                    classes.forEach(cls => {
                        if (cls.teacher === id) {
                            cls.teacher = null;
                        }
                    });
                });
                
                selectedTeachers.clear();
                saveDataWithNotification(`Guru yang dipilih berhasil dihapus!`);
                renderTeachers();
                renderClasses();
                populateTeacherSelects();
            }
        }

        function deleteAllTeachers() {
            if (teachers.length === 0) return;
            
            if (confirm(`Apakah Anda yakin ingin menghapus SEMUA ${teachers.length} guru? Tindakan ini tidak dapat dibatalkan!`)) {
                teachers = [];
                // Remove all teacher assignments from classes
                classes.forEach(cls => {
                    cls.teacher = null;
                });
                selectedTeachers.clear();
                saveDataWithNotification('Semua data guru berhasil dihapus!');
                renderTeachers();
                renderClasses();
                populateTeacherSelects();
            }
        }

        function openAddTeacherModal() {
            document.getElementById('addTeacherModal').classList.remove('hidden');
            document.getElementById('addTeacherModal').classList.add('flex');
        }

        function closeAddTeacherModal() {
            document.getElementById('addTeacherModal').classList.add('hidden');
            document.getElementById('addTeacherModal').classList.remove('flex');
            document.getElementById('newTeacherName').value = '';
            document.getElementById('newTeacherNIP').value = '';
        }

        function addTeacher(event) {
            event.preventDefault();
            const name = document.getElementById('newTeacherName').value.trim();
            const nip = document.getElementById('newTeacherNIP').value.trim();
            
            if (!name || !nip) {
                showNotification('Semua field harus diisi', 'error');
                return;
            }
            
            const newTeacher = {
                id: Date.now(),
                name: name,
                nip: nip
            };
            
            teachers.push(newTeacher);
            saveDataWithNotification('Guru berhasil ditambahkan!');
            renderTeachers();
            populateTeacherSelects();
            closeAddTeacherModal();
        }

        function editTeacher(id) {
            const teacher = teachers.find(t => t.id === id);
            if (!teacher) return;
            
            // Populate edit form
            document.getElementById('editTeacherId').value = teacher.id;
            document.getElementById('editTeacherName').value = teacher.name;
            document.getElementById('editTeacherNIP').value = teacher.nip;
            
            // Show modal
            document.getElementById('editTeacherModal').classList.remove('hidden');
            document.getElementById('editTeacherModal').classList.add('flex');
        }

        function closeEditTeacherModal() {
            document.getElementById('editTeacherModal').classList.add('hidden');
            document.getElementById('editTeacherModal').classList.remove('flex');
        }

        function updateTeacher(event) {
            event.preventDefault();
            const id = parseInt(document.getElementById('editTeacherId').value);
            const name = document.getElementById('editTeacherName').value.trim();
            const nip = document.getElementById('editTeacherNIP').value.trim();
            
            if (!name || !nip) {
                showNotification('Nama dan NIP harus diisi', 'error');
                return;
            }
            
            const teacherIndex = teachers.findIndex(t => t.id === id);
            if (teacherIndex !== -1) {
                teachers[teacherIndex] = {
                    ...teachers[teacherIndex],
                    name: name,
                    nip: nip
                };
                
                saveDataWithNotification('Data guru berhasil diperbarui!');
                renderTeachers();
                populateTeacherSelects();
                renderClasses();
                closeEditTeacherModal();
            }
        }

        function deleteTeacher(id) {
            const teacher = teachers.find(t => t.id === id);
            if (confirm(`Apakah Anda yakin ingin menghapus guru "${teacher.name}"?`)) {
                // Check if teacher is assigned to any class
                const assignedClasses = classes.filter(c => c.teacher === id);
                if (assignedClasses.length > 0) {
                    const classNames = assignedClasses.map(c => c.name).join(', ');
                    if (!confirm(`Guru ini masih menjadi wali kelas: ${classNames}. Hapus tetap?`)) {
                        return;
                    }
                    // Remove teacher assignment from classes
                    assignedClasses.forEach(cls => {
                        const classIndex = classes.findIndex(c => c.id === cls.id);
                        if (classIndex !== -1) {
                            classes[classIndex].teacher = null;
                        }
                    });
                }
                
                teachers = teachers.filter(t => t.id !== id);
                saveDataWithNotification('Guru berhasil dihapus!');
                renderTeachers();
                renderClasses();
                populateTeacherSelects();
            }
        }

        // Subject Management Functions
        function renderSubjects() {
            const container = document.getElementById('subjectsList');
            container.innerHTML = '';
            
            if (subjects.length === 0) {
                container.innerHTML = '<p class="text-gray-500 text-center py-4">Belum ada mata pelajaran</p>';
                return;
            }
            
            subjects.forEach(subject => {
                const div = document.createElement('div');
                div.className = 'flex items-center justify-between p-3 bg-gray-50 rounded-lg hover:bg-gray-100 transition-colors';
                div.innerHTML = `
                    <div class="flex items-center space-x-3">
                        <input type="checkbox" class="subject-checkbox rounded" value="${subject.id}" onchange="updateSubjectSelection()" ${selectedSubjects.has(subject.id) ? 'checked' : ''}>
                        <div>
                            <div class="font-medium text-gray-800">${subject.name}</div>
                            <div class="text-sm text-gray-500">Kode: ${subject.code}</div>
                        </div>
                    </div>
                    <div class="flex space-x-2">
                        <button onclick="editSubject(${subject.id})" class="text-blue-600 hover:text-blue-800" title="Edit">
                            <i class="fas fa-edit"></i>
                        </button>
                        <button onclick="deleteSubject(${subject.id})" class="text-red-600 hover:text-red-800" title="Hapus">
                            <i class="fas fa-trash"></i>
                        </button>
                    </div>
                `;
                container.appendChild(div);
            });
            
            updateSubjectSelection();
        }

        function updateSubjectSelection() {
            const checkboxes = document.querySelectorAll('.subject-checkbox');
            selectedSubjects.clear();
            
            checkboxes.forEach(checkbox => {
                if (checkbox.checked) {
                    selectedSubjects.add(parseInt(checkbox.value));
                }
            });
            
            const count = selectedSubjects.size;
            document.getElementById('selectedSubjectsCount').textContent = count;
            
            const bulkActions = document.getElementById('subjectBulkActions');
            if (count > 0) {
                bulkActions.classList.remove('hidden');
            } else {
                bulkActions.classList.add('hidden');
            }
        }

        function selectAllSubjects() {
            const checkboxes = document.querySelectorAll('.subject-checkbox');
            checkboxes.forEach(checkbox => {
                checkbox.checked = true;
            });
            updateSubjectSelection();
        }

        function deselectAllSubjects() {
            const checkboxes = document.querySelectorAll('.subject-checkbox');
            checkboxes.forEach(checkbox => {
                checkbox.checked = false;
            });
            updateSubjectSelection();
        }

        function deleteSelectedSubjects() {
            if (selectedSubjects.size === 0) return;
            
            if (confirm(`Apakah Anda yakin ingin menghapus ${selectedSubjects.size} mata pelajaran yang dipilih?`)) {
                selectedSubjects.forEach(id => {
                    subjects = subjects.filter(s => s.id !== id);
                });
                
                selectedSubjects.clear();
                saveDataWithNotification(`Mata pelajaran yang dipilih berhasil dihapus!`);
                renderSubjects();
            }
        }

        function deleteAllSubjects() {
            if (subjects.length === 0) return;
            
            if (confirm(`Apakah Anda yakin ingin menghapus SEMUA ${subjects.length} mata pelajaran? Tindakan ini tidak dapat dibatalkan!`)) {
                subjects = [];
                selectedSubjects.clear();
                saveDataWithNotification('Semua mata pelajaran berhasil dihapus!');
                renderSubjects();
            }
        }

        function openAddSubjectModal() {
            document.getElementById('addSubjectModal').classList.remove('hidden');
            document.getElementById('addSubjectModal').classList.add('flex');
        }

        function closeAddSubjectModal() {
            document.getElementById('addSubjectModal').classList.add('hidden');
            document.getElementById('addSubjectModal').classList.remove('flex');
            document.getElementById('newSubjectName').value = '';
            document.getElementById('newSubjectCode').value = '';
        }

        function addSubject(event) {
            event.preventDefault();
            const name = document.getElementById('newSubjectName').value.trim();
            const code = document.getElementById('newSubjectCode').value.trim();
            
            if (!name || !code) {
                showNotification('Semua field harus diisi', 'error');
                return;
            }
            
            const newSubject = {
                id: Date.now(),
                name: name,
                code: code
            };
            
            subjects.push(newSubject);
            
            // Initialize grades for all existing students for this new subject
            students.forEach(student => {
                if (!grades[student.id]) grades[student.id] = {};
                grades[student.id][newSubject.id] = {
                    attendance: 0,
                    assignment: 0,
                    practice: 0,
                    midterm: 0,
                    finalexam: 0
                };
            });
            
            saveDataWithNotification('Mata pelajaran berhasil ditambahkan!');
            renderSubjects();
            populateSubjectSelect();
            closeAddSubjectModal();
        }

        function editSubject(id) {
            const subject = subjects.find(s => s.id === id);
            if (!subject) return;
            
            // Populate edit form
            document.getElementById('editSubjectId').value = subject.id;
            document.getElementById('editSubjectName').value = subject.name;
            document.getElementById('editSubjectCode').value = subject.code;
            
            // Show modal
            document.getElementById('editSubjectModal').classList.remove('hidden');
            document.getElementById('editSubjectModal').classList.add('flex');
        }

        function closeEditSubjectModal() {
            document.getElementById('editSubjectModal').classList.add('hidden');
            document.getElementById('editSubjectModal').classList.remove('flex');
        }

        function updateSubject(event) {
            event.preventDefault();
            const id = parseInt(document.getElementById('editSubjectId').value);
            const name = document.getElementById('editSubjectName').value.trim();
            const code = document.getElementById('editSubjectCode').value.trim();
            
            if (!name || !code) {
                showNotification('Nama dan kode mata pelajaran harus diisi', 'error');
                return;
            }
            
            const subjectIndex = subjects.findIndex(s => s.id === id);
            if (subjectIndex !== -1) {
                subjects[subjectIndex] = {
                    ...subjects[subjectIndex],
                    name: name,
                    code: code
                };
                
                saveDataWithNotification('Mata pelajaran berhasil diperbarui!');
                renderSubjects();
                populateSubjectSelect();
                closeEditSubjectModal();
            }
        }

        function deleteSubject(id) {
            const subject = subjects.find(s => s.id === id);
            if (confirm(`Apakah Anda yakin ingin menghapus mata pelajaran "${subject.name}"? Semua nilai untuk mata pelajaran ini akan ikut terhapus!`)) {
                // Remove subject from all student grades
                students.forEach(student => {
                    if (grades[student.id] && grades[student.id][id]) {
                        delete grades[student.id][id];
                    }
                });
                
                subjects = subjects.filter(s => s.id !== id);
                saveDataWithNotification('Mata pelajaran berhasil dihapus!');
                renderSubjects();
                populateSubjectSelect();
                renderStudentsTable();
                updateDashboard();
            }
        }

        // Class Management Functions
        function renderClasses() {
            const container = document.getElementById('classesList');
            container.innerHTML = '';
            
            if (classes.length === 0) {
                container.innerHTML = '<p class="text-gray-500 text-center py-4">Belum ada data kelas</p>';
                return;
            }
            
            classes.forEach(cls => {
                const teacher = teachers.find(t => t.id === cls.teacher);
                const studentCount = students.filter(s => s.class === cls.id).length;
                
                const div = document.createElement('div');
                div.className = 'bg-gray-50 p-4 rounded-lg hover:bg-gray-100 transition-colors';
                div.innerHTML = `
                    <div class="flex items-center justify-between mb-2">
                        <div class="flex items-center space-x-3">
                            <input type="checkbox" class="class-checkbox rounded" value="${cls.id}" onchange="updateClassSelection()" ${selectedClasses.has(cls.id) ? 'checked' : ''}>
                            <h4 class="font-semibold text-gray-800">${cls.name}</h4>
                        </div>
                        <div class="flex space-x-2">
                            <button onclick="editClass(${cls.id})" class="text-blue-600 hover:text-blue-800" title="Edit">
                                <i class="fas fa-edit"></i>
                            </button>
                            <button onclick="deleteClass(${cls.id})" class="text-red-600 hover:text-red-800" title="Hapus">
                                <i class="fas fa-trash"></i>
                            </button>
                        </div>
                    </div>
                    <div class="text-sm text-gray-600 ml-8">
                        <div>Wali Kelas: ${teacher?.name || 'Belum ditentukan'}</div>
                        <div>Siswa: ${studentCount}/${cls.capacity}</div>
                        <div class="w-full bg-gray-200 rounded-full h-2 mt-2">
                            <div class="bg-blue-600 h-2 rounded-full" style="width: ${(studentCount/cls.capacity)*100}%"></div>
                        </div>
                    </div>
                `;
                container.appendChild(div);
            });
            
            updateClassSelection();
        }

        function updateClassSelection() {
            const checkboxes = document.querySelectorAll('.class-checkbox');
            selectedClasses.clear();
            
            checkboxes.forEach(checkbox => {
                if (checkbox.checked) {
                    selectedClasses.add(parseInt(checkbox.value));
                }
            });
            
            const count = selectedClasses.size;
            document.getElementById('selectedClassesCount').textContent = count;
            
            const bulkActions = document.getElementById('classBulkActions');
            if (count > 0) {
                bulkActions.classList.remove('hidden');
            } else {
                bulkActions.classList.add('hidden');
            }
        }

        function selectAllClasses() {
            const checkboxes = document.querySelectorAll('.class-checkbox');
            checkboxes.forEach(checkbox => {
                checkbox.checked = true;
            });
            updateClassSelection();
        }

        function deselectAllClasses() {
            const checkboxes = document.querySelectorAll('.class-checkbox');
            checkboxes.forEach(checkbox => {
                checkbox.checked = false;
            });
            updateClassSelection();
        }

        function deleteSelectedClasses() {
            if (selectedClasses.size === 0) return;
            
            if (confirm(`Apakah Anda yakin ingin menghapus ${selectedClasses.size} kelas yang dipilih?`)) {
                selectedClasses.forEach(id => {
                    // Remove class assignment from students
                    students.forEach(student => {
                        if (student.class === id) {
                            student.class = null;
                        }
                    });
                    classes = classes.filter(c => c.id !== id);
                });
                
                selectedClasses.clear();
                saveDataWithNotification(`Kelas yang dipilih berhasil dihapus!`);
                renderClasses();
                renderStudentsTable();
                populateClassSelects();
                populateStudentSelect();
                updateDashboard();
            }
        }

        function deleteAllClasses() {
            if (classes.length === 0) return;
            
            if (confirm(`Apakah Anda yakin ingin menghapus SEMUA ${classes.length} kelas? Tindakan ini tidak dapat dibatalkan!`)) {
                // Remove all class assignments from students
                students.forEach(student => {
                    student.class = null;
                });
                classes = [];
                selectedClasses.clear();
                saveDataWithNotification('Semua kelas berhasil dihapus!');
                renderClasses();
                renderStudentsTable();
                populateClassSelects();
                populateStudentSelect();
                updateDashboard();
            }
        }

        function openAddClassModal() {
            populateTeacherSelects();
            document.getElementById('addClassModal').classList.remove('hidden');
            document.getElementById('addClassModal').classList.add('flex');
        }

        function closeAddClassModal() {
            document.getElementById('addClassModal').classList.add('hidden');
            document.getElementById('addClassModal').classList.remove('flex');
            document.getElementById('newClassName').value = '';
            document.getElementById('newClassTeacher').value = '';
            document.getElementById('newClassCapacity').value = '30';
        }

        function addClass(event) {
            event.preventDefault();
            const name = document.getElementById('newClassName').value.trim();
            const teacher = parseInt(document.getElementById('newClassTeacher').value) || null;
            const capacity = parseInt(document.getElementById('newClassCapacity').value);
            
            if (!name || !capacity) {
                showNotification('Nama kelas dan kapasitas harus diisi', 'error');
                return;
            }
            
            const newClass = {
                id: Date.now(),
                name: name,
                teacher: teacher,
                capacity: capacity
            };
            
            classes.push(newClass);
            saveDataWithNotification('Kelas berhasil ditambahkan!');
            renderClasses();
            populateClassSelects();
            closeAddClassModal();
        }

        function editClass(id) {
            const cls = classes.find(c => c.id === id);
            if (!cls) return;
            
            // Populate edit form
            document.getElementById('editClassId').value = cls.id;
            document.getElementById('editClassName').value = cls.name;
            document.getElementById('editClassTeacher').value = cls.teacher || '';
            document.getElementById('editClassCapacity').value = cls.capacity;
            
            // Populate teacher options
            populateEditTeacherSelects();
            
            // Show modal
            document.getElementById('editClassModal').classList.remove('hidden');
            document.getElementById('editClassModal').classList.add('flex');
        }

        function closeEditClassModal() {
            document.getElementById('editClassModal').classList.add('hidden');
            document.getElementById('editClassModal').classList.remove('flex');
        }

        function updateClass(event) {
            event.preventDefault();
            const id = parseInt(document.getElementById('editClassId').value);
            const name = document.getElementById('editClassName').value.trim();
            const teacher = parseInt(document.getElementById('editClassTeacher').value) || null;
            const capacity = parseInt(document.getElementById('editClassCapacity').value);
            
            if (!name || !capacity) {
                showNotification('Nama kelas dan kapasitas harus diisi', 'error');
                return;
            }
            
            const classIndex = classes.findIndex(c => c.id === id);
            if (classIndex !== -1) {
                classes[classIndex] = {
                    ...classes[classIndex],
                    name: name,
                    teacher: teacher,
                    capacity: capacity
                };
                
                saveDataWithNotification('Kelas berhasil diperbarui!');
                renderClasses();
                populateClassSelects();
                renderStudentsTable();
                populateStudentSelect();
                closeEditClassModal();
            }
        }

        function deleteClass(id) {
            const cls = classes.find(c => c.id === id);
            if (confirm(`Apakah Anda yakin ingin menghapus kelas "${cls.name}"?`)) {
                // Check if there are students in this class
                const studentsInClass = students.filter(s => s.class === id);
                if (studentsInClass.length > 0) {
                    if (!confirm(`Kelas ini memiliki ${studentsInClass.length} siswa. Hapus tetap?`)) {
                        return;
                    }
                    // Remove class assignment from students
                    studentsInClass.forEach(student => {
                        const studentIndex = students.findIndex(s => s.id === student.id);
                        if (studentIndex !== -1) {
                            students[studentIndex].class = null;
                        }
                    });
                }
                
                classes = classes.filter(c => c.id !== id);
                saveDataWithNotification('Kelas berhasil dihapus!');
                renderClasses();
                populateClassSelects();
                renderStudentsTable();
                populateStudentSelect();
                updateDashboard();
            }
        }

        // Populate Functions
        function populateSubjectSelect() {
            const select = document.getElementById('subjectSelect');
            if (select) {
                const currentValue = select.value;
                select.innerHTML = '<option value="">Pilih mata pelajaran...</option>';
                
                subjects.forEach(subject => {
                    const option = document.createElement('option');
                    option.value = subject.id;
                    option.textContent = `${subject.name} (${subject.code})`;
                    if (subject.id == currentValue) option.selected = true;
                    select.appendChild(option);
                });
            }
        }

        function populateClassSelects() {
            const selects = [
                'studentsClassFilter',
                'gradesClassFilter', 
                'reportsClassFilter'
            ];
            
            selects.forEach(selectId => {
                const select = document.getElementById(selectId);
                if (select) {
                    const currentValue = select.value;
                    select.innerHTML = '<option value="">Semua Kelas</option>';
                    
                    classes.forEach(cls => {
                        const option = document.createElement('option');
                        option.value = cls.id;
                        option.textContent = cls.name;
                        if (cls.id == currentValue) option.selected = true;
                        select.appendChild(option);
                    });
                }
            });

            // For add/edit student modals
            const newStudentClass = document.getElementById('newStudentClass');
            if (newStudentClass) {
                newStudentClass.innerHTML = '<option value="">Pilih kelas</option>';
                classes.forEach(cls => {
                    const option = document.createElement('option');
                    option.value = cls.id;
                    option.textContent = cls.name;
                    newStudentClass.appendChild(option);
                });
            }
        }

        function populateReportsSubjectSelect() {
            const select = document.getElementById('reportsSubjectFilter');
            if (select) {
                const currentValue = select.value;
                select.innerHTML = '<option value="">Semua Mata Pelajaran</option>';
                
                subjects.forEach(subject => {
                    const option = document.createElement('option');
                    option.value = subject.id;
                    option.textContent = `${subject.name} (${subject.code})`;
                    if (subject.id == currentValue) option.selected = true;
                    select.appendChild(option);
                });
            }
        }

        function populateEditClassSelects() {
            const select = document.getElementById('editStudentClass');
            if (select) {
                select.innerHTML = '<option value="">Pilih kelas</option>';
                classes.forEach(cls => {
                    const option = document.createElement('option');
                    option.value = cls.id;
                    option.textContent = cls.name;
                    select.appendChild(option);
                });
            }
        }

        function populateTeacherSelects() {
            const selects = ['newClassTeacher'];
            
            selects.forEach(selectId => {
                const select = document.getElementById(selectId);
                if (select) {
                    select.innerHTML = '<option value="">Pilih wali kelas</option>';
                    teachers.forEach(teacher => {
                        const option = document.createElement('option');
                        option.value = teacher.id;
                        option.textContent = teacher.name;
                        select.appendChild(option);
                    });
                }
            });
        }

        function populateEditTeacherSelects() {
            const select = document.getElementById('editClassTeacher');
            if (select) {
                select.innerHTML = '<option value="">Pilih wali kelas</option>';
                teachers.forEach(teacher => {
                    const option = document.createElement('option');
                    option.value = teacher.id;
                    option.textContent = teacher.name;
                    select.appendChild(option);
                });
            }
        }

        // Excel Functions
        function downloadExcelTemplate() {
            const wb = XLSX.utils.book_new();
            const wsData = [
                ['Nama Siswa', 'Kelas', 'Jenis Kelamin'],
                ['Contoh: Ahmad Rizki', 'X IPA 1', 'Laki-laki'],
                ['Contoh: Siti Nurhaliza', 'X IPA 2', 'Perempuan']
            ];
            
            const ws = XLSX.utils.aoa_to_sheet(wsData);
            XLSX.utils.book_append_sheet(wb, ws, 'Template Siswa');
            XLSX.writeFile(wb, 'Template_Data_Siswa.xlsx');
            showNotification('Template Excel berhasil didownload!');
        }

        function importExcel() {
            const file = document.getElementById('excelFile').files[0];
            if (!file) return;
            
            const reader = new FileReader();
            reader.onload = function(e) {
                try {
                    const data = new Uint8Array(e.target.result);
                    const workbook = XLSX.read(data, {type: 'array'});
                    const sheetName = workbook.SheetNames[0];
                    const worksheet = workbook.Sheets[sheetName];
                    const jsonData = XLSX.utils.sheet_to_json(worksheet, {header: 1});
                    
                    // Skip header row
                    const rows = jsonData.slice(1);
                    let imported = 0;
                    
                    rows.forEach(row => {
                        if (row[0] && row[1] && row[2]) { // Check if required fields exist
                            const name = row[0].toString().trim();
                            const className = row[1].toString().trim();
                            const gender = row[2].toString().trim();
                            
                            // Find or create class
                            let classObj = classes.find(c => c.name === className);
                            if (!classObj) {
                                classObj = {
                                    id: Date.now() + Math.random(),
                                    name: className,
                                    teacher: null,
                                    capacity: 30
                                };
                                classes.push(classObj);
                            }
                            
                            // Add student
                            const newStudent = {
                                id: Date.now() + Math.random(),
                                name: name,
                                class: classObj.id,
                                gender: gender
                            };
                            
                            students.push(newStudent);
                            grades[newStudent.id] = {};
                            // Initialize grades for all subjects
                            subjects.forEach(subject => {
                                grades[newStudent.id][subject.id] = {
                                    attendance: 0,
                                    assignment: 0,
                                    practice: 0,
                                    midterm: 0,
                                    finalexam: 0
                                };
                            });
                            imported++;
                        }
                    });
                    
                    saveDataWithNotification(`Berhasil import ${imported} siswa dari Excel!`);
                    renderStudentsTable();
                    populateClassSelects();
                    populateStudentSelect();
                    updateDashboard();
                    
                } catch (error) {
                    showNotification('Error membaca file Excel: ' + error.message, 'error');
                }
            };
            reader.readAsArrayBuffer(file);
        }

        // Report Functions
        function updateReportPreview() {
            const selectedClass = document.getElementById('reportsClassFilter')?.value;
            const selectedSubject = document.getElementById('reportsSubjectFilter')?.value;
            const selectedPeriod = document.getElementById('reportsPeriodFilter')?.value;
            const reportTitle = document.getElementById('reportTitle')?.value || 'Laporan Nilai Siswa';
            
            let reportStudents = students;
            
            // Filter by class
            if (selectedClass) {
                reportStudents = reportStudents.filter(s => s.class == selectedClass);
            }
            
            const container = document.getElementById('reportPreview');
            
            if (reportStudents.length === 0) {
                container.innerHTML = `
                    <div class="text-center py-12">
                        <i class="fas fa-exclamation-triangle text-yellow-400 text-4xl mb-4"></i>
                        <p class="text-gray-600">Tidak ada data siswa untuk filter yang dipilih</p>
                    </div>
                `;
                return;
            }
            
            // Get subject info
            const subjectInfo = selectedSubject ? subjects.find(s => s.id == selectedSubject) : null;
            const className = selectedClass ? classes.find(c => c.id == selectedClass)?.name : 'Semua Kelas';
            
            // Calculate statistics
            let totalGrades = [];
            reportStudents.forEach(student => {
                if (selectedSubject) {
                    const grade = calculateStudentFinalGrade(student.id, parseInt(selectedSubject));
                    totalGrades.push(grade);
                } else {
                    const grade = calculateStudentFinalGrade(student.id);
                    totalGrades.push(grade);
                }
            });
            
            const avgGrade = totalGrades.reduce((sum, grade) => sum + grade, 0) / totalGrades.length;
            const excellentCount = totalGrades.filter(g => g >= 90).length;
            const goodCount = totalGrades.filter(g => g >= 80 && g < 90).length;
            const fairCount = totalGrades.filter(g => g >= 70 && g < 80).length;
            const poorCount = totalGrades.filter(g => g < 70).length;
            
            // Generate preview
            let previewHTML = `
                <div class="space-y-6">
                    <!-- Report Header -->
                    <div class="text-center border-b pb-4">
                        <h3 class="text-xl font-bold text-gray-800">${schoolSettings.name}</h3>
                        <h4 class="text-lg font-semibold text-gray-700 mt-2">${reportTitle}</h4>
                        <div class="text-sm text-gray-600 mt-2">
                            <p>Kelas: ${className}</p>
                            <p>Mata Pelajaran: ${subjectInfo ? `${subjectInfo.name} (${subjectInfo.code})` : 'Semua Mata Pelajaran'}</p>
                            <p>Periode: ${selectedPeriod === 'semester1' ? 'Semester 1' : selectedPeriod === 'semester2' ? 'Semester 2' : 'Tahunan'}</p>
                            <p>Tanggal: ${new Date().toLocaleDateString('id-ID')}</p>
                        </div>
                    </div>
                    
                    <!-- Statistics -->
                    <div class="grid grid-cols-2 md:grid-cols-4 gap-4">
                        <div class="bg-green-50 p-4 rounded-lg text-center">
                            <div class="text-2xl font-bold text-green-600">${excellentCount}</div>
                            <div class="text-sm text-green-700">Sangat Baik (90-100)</div>
                        </div>
                        <div class="bg-blue-50 p-4 rounded-lg text-center">
                            <div class="text-2xl font-bold text-blue-600">${goodCount}</div>
                            <div class="text-sm text-blue-700">Baik (80-89)</div>
                        </div>
                        <div class="bg-yellow-50 p-4 rounded-lg text-center">
                            <div class="text-2xl font-bold text-yellow-600">${fairCount}</div>
                            <div class="text-sm text-yellow-700">Cukup (70-79)</div>
                        </div>
                        <div class="bg-red-50 p-4 rounded-lg text-center">
                            <div class="text-2xl font-bold text-red-600">${poorCount}</div>
                            <div class="text-sm text-red-700">Perlu Perbaikan (<70)</div>
                        </div>
                    </div>
                    
                    <!-- Average Grade -->
                    <div class="bg-gray-50 p-4 rounded-lg text-center">
                        <div class="text-3xl font-bold text-gray-800">${avgGrade.toFixed(1)}</div>
                        <div class="text-sm text-gray-600">Rata-rata Nilai</div>
                    </div>
                    
                    <!-- Student List Preview -->
                    <div>
                        <h5 class="font-semibold text-gray-800 mb-3">Daftar Siswa (${reportStudents.length} siswa)</h5>
                        <div class="space-y-2 max-h-64 overflow-y-auto">
            `;
            
            // Show first 10 students
            const previewStudents = reportStudents.slice(0, 10);
            previewStudents.forEach((student, index) => {
                const studentClass = classes.find(c => c.id === student.class)?.name || '';
                const finalGrade = selectedSubject ? 
                    calculateStudentFinalGrade(student.id, parseInt(selectedSubject)) : 
                    calculateStudentFinalGrade(student.id);
                
                previewHTML += `
                    <div class="flex justify-between items-center p-3 bg-white rounded border">
                        <div>
                            <span class="font-medium">${index + 1}. ${student.name}</span>
                            <span class="text-sm text-gray-500 ml-2">(${studentClass})</span>
                        </div>
                        <div class="text-right">
                            <span class="px-2 py-1 rounded text-sm font-medium ${getGradeColor(finalGrade)}">${finalGrade.toFixed(1)}</span>
                            <div class="text-xs text-gray-500">${getGradeStatus(finalGrade)}</div>
                        </div>
                    </div>
                `;
            });
            
            if (reportStudents.length > 10) {
                previewHTML += `
                    <div class="text-center p-3 bg-gray-100 rounded border">
                        <span class="text-gray-600">... dan ${reportStudents.length - 10} siswa lainnya</span>
                    </div>
                `;
            }
            
            previewHTML += `
                        </div>
                    </div>
                </div>
            `;
            
            container.innerHTML = previewHTML;
        }

        function downloadExcel() {
            const selectedClass = document.getElementById('reportsClassFilter')?.value;
            const selectedSubject = document.getElementById('reportsSubjectFilter')?.value;
            const selectedPeriod = document.getElementById('reportsPeriodFilter')?.value;
            const reportTitle = document.getElementById('reportTitle')?.value || 'Laporan Nilai Siswa';
            
            let reportStudents = students;
            
            if (selectedClass) {
                reportStudents = students.filter(s => s.class == selectedClass);
            }
            
            const wb = XLSX.utils.book_new();
            
            if (selectedSubject) {
                // Single subject report
                const subject = subjects.find(s => s.id == selectedSubject);
                const reportData = [
                    ['No', 'Nama Siswa', 'Kelas', 'Jenis Kelamin', 'Kehadiran', 'Tugas', 'Praktek', 'UTS', 'UAS', 'Nilai Akhir', 'Status']
                ];
                
                reportStudents.forEach((student, index) => {
                    const className = classes.find(c => c.id === student.class)?.name || '';
                    const studentGrades = (grades[student.id] && grades[student.id][selectedSubject]) || {};
                    const finalGrade = calculateStudentFinalGrade(student.id, parseInt(selectedSubject));
                    
                    reportData.push([
                        index + 1,
                        student.name,
                        className,
                        student.gender,
                        studentGrades.attendance || 0,
                        studentGrades.assignment || 0,
                        studentGrades.practice || 0,
                        studentGrades.midterm || 0,
                        studentGrades.finalexam || 0,
                        finalGrade.toFixed(1),
                        getGradeStatus(finalGrade)
                    ]);
                });
                
                const ws = XLSX.utils.aoa_to_sheet(reportData);
                XLSX.utils.book_append_sheet(wb, ws, subject.name);
                
            } else {
                // All subjects report - create summary sheet and individual subject sheets
                
                // Summary sheet with all subjects
                const summaryData = [
                    ['No', 'Nama Siswa', 'Kelas', 'Jenis Kelamin', ...subjects.map(s => s.name), 'Rata-rata', 'Status']
                ];
                
                reportStudents.forEach((student, index) => {
                    const className = classes.find(c => c.id === student.class)?.name || '';
                    const subjectGrades = subjects.map(subject => {
                        return calculateStudentFinalGrade(student.id, subject.id).toFixed(1);
                    });
                    const avgGrade = calculateStudentFinalGrade(student.id);
                    
                    summaryData.push([
                        index + 1,
                        student.name,
                        className,
                        student.gender,
                        ...subjectGrades,
                        avgGrade.toFixed(1),
                        getGradeStatus(avgGrade)
                    ]);
                });
                
                const summaryWs = XLSX.utils.aoa_to_sheet(summaryData);
                XLSX.utils.book_append_sheet(wb, summaryWs, 'Ringkasan');
                
                // Individual subject sheets
                subjects.forEach(subject => {
                    const subjectData = [
                        ['No', 'Nama Siswa', 'Kelas', 'Jenis Kelamin', 'Kehadiran', 'Tugas', 'Praktek', 'UTS', 'UAS', 'Nilai Akhir', 'Status']
                    ];
                    
                    reportStudents.forEach((student, index) => {
                        const className = classes.find(c => c.id === student.class)?.name || '';
                        const studentGrades = (grades[student.id] && grades[student.id][subject.id]) || {};
                        const finalGrade = calculateStudentFinalGrade(student.id, subject.id);
                        
                        subjectData.push([
                            index + 1,
                            student.name,
                            className,
                            student.gender,
                            studentGrades.attendance || 0,
                            studentGrades.assignment || 0,
                            studentGrades.practice || 0,
                            studentGrades.midterm || 0,
                            studentGrades.finalexam || 0,
                            finalGrade.toFixed(1),
                            getGradeStatus(finalGrade)
                        ]);
                    });
                    
                    const subjectWs = XLSX.utils.aoa_to_sheet(subjectData);
                    XLSX.utils.book_append_sheet(wb, subjectWs, subject.name);
                });
            }
            
            const fileName = selectedSubject ? 
                `${reportTitle}_${subjects.find(s => s.id == selectedSubject)?.name}.xlsx` : 
                `${reportTitle}_Lengkap.xlsx`;
                
            XLSX.writeFile(wb, fileName);
            showNotification('Laporan Excel berhasil didownload!');
        }

        function downloadPDF() {
            const { jsPDF } = window.jspdf;
            const doc = new jsPDF();
            
            const selectedClass = document.getElementById('reportsClassFilter')?.value;
            const selectedSubject = document.getElementById('reportsSubjectFilter')?.value;
            const selectedPeriod = document.getElementById('reportsPeriodFilter')?.value;
            const title = document.getElementById('reportTitle').value || 'Laporan Nilai Siswa';
            
            let reportStudents = students;
            if (selectedClass) {
                reportStudents = students.filter(s => s.class == selectedClass);
            }
            
            const subjectInfo = selectedSubject ? subjects.find(s => s.id == selectedSubject) : null;
            const className = selectedClass ? classes.find(c => c.id == selectedClass)?.name : 'Semua Kelas';
            const periodText = selectedPeriod === 'semester1' ? 'Semester 1' : selectedPeriod === 'semester2' ? 'Semester 2' : 'Tahunan';
            
            // Header
            doc.setFontSize(16);
            doc.text(schoolSettings.name, 20, 20);
            doc.setFontSize(14);
            doc.text(title, 20, 30);
            doc.setFontSize(10);
            doc.text(`Kelas: ${className}`, 20, 40);
            doc.text(`Mata Pelajaran: ${subjectInfo ? `${subjectInfo.name} (${subjectInfo.code})` : 'Semua Mata Pelajaran'}`, 20, 48);
            doc.text(`Periode: ${periodText}`, 20, 56);
            doc.text(`Tanggal: ${new Date().toLocaleDateString('id-ID')}`, 20, 64);
            
            // Table
            let y = 80;
            doc.setFontSize(9);
            doc.text('No', 20, y);
            doc.text('Nama Siswa', 30, y);
            doc.text('Kelas', 80, y);
            doc.text('Nilai Akhir', 110, y);
            doc.text('Status', 140, y);
            
            y += 8;
            reportStudents.forEach((student, index) => {
                const studentClass = classes.find(c => c.id === student.class)?.name || '';
                const finalGrade = selectedSubject ? 
                    calculateStudentFinalGrade(student.id, parseInt(selectedSubject)) : 
                    calculateStudentFinalGrade(student.id);
                
                doc.text((index + 1).toString(), 20, y);
                doc.text(student.name.substring(0, 20), 30, y);
                doc.text(studentClass, 80, y);
                doc.text(finalGrade.toFixed(1), 110, y);
                doc.text(getGradeStatus(finalGrade), 140, y);
                y += 6;
                
                if (y > 280) {
                    doc.addPage();
                    y = 20;
                }
            });
            
            const fileName = selectedSubject ? 
                `${title}_${subjectInfo.name}.pdf` : 
                `${title}_Lengkap.pdf`;
                
            doc.save(fileName);
            showNotification('Laporan PDF berhasil didownload!');
        }

        function printReport() {
            const selectedClass = document.getElementById('reportsClassFilter')?.value;
            const selectedSubject = document.getElementById('reportsSubjectFilter')?.value;
            const selectedPeriod = document.getElementById('reportsPeriodFilter')?.value;
            const title = document.getElementById('reportTitle').value || 'Laporan Nilai Siswa';
            
            let reportStudents = students;
            if (selectedClass) {
                reportStudents = students.filter(s => s.class == selectedClass);
            }
            
            const subjectInfo = selectedSubject ? subjects.find(s => s.id == selectedSubject) : null;
            const className = selectedClass ? classes.find(c => c.id == selectedClass)?.name : 'Semua Kelas';
            const periodText = selectedPeriod === 'semester1' ? 'Semester 1' : selectedPeriod === 'semester2' ? 'Semester 2' : 'Tahunan';
            
            let printContent = `
                <html>
                <head>
                    <title>${title}</title>
                    <style>
                        body { font-family: Arial, sans-serif; margin: 20px; }
                        .header { text-align: center; margin-bottom: 30px; }
                        table { width: 100%; border-collapse: collapse; }
                        th, td { border: 1px solid #ddd; padding: 8px; text-align: left; }
                        th { background-color: #f2f2f2; }
                        .text-center { text-align: center; }
                        .info-table { margin-bottom: 20px; }
                        .info-table td { border: none; padding: 4px 8px; }
                    </style>
                </head>
                <body>
                    <div class="header">
                        <h2>${schoolSettings.name}</h2>
                        <h3>${title}</h3>
                    </div>
                    <table class="info-table">
                        <tr>
                            <td><strong>Kelas:</strong></td>
                            <td>${className}</td>
                        </tr>
                        <tr>
                            <td><strong>Mata Pelajaran:</strong></td>
                            <td>${subjectInfo ? `${subjectInfo.name} (${subjectInfo.code})` : 'Semua Mata Pelajaran'}</td>
                        </tr>
                        <tr>
                            <td><strong>Periode:</strong></td>
                            <td>${periodText}</td>
                        </tr>
                        <tr>
                            <td><strong>Tanggal:</strong></td>
                            <td>${new Date().toLocaleDateString('id-ID')}</td>
                        </tr>
                    </table>
                    <table>
                        <thead>
                            <tr>
                                <th>No</th>
                                <th>Nama Siswa</th>
                                <th>Kelas</th>
                                <th>Jenis Kelamin</th>
                                <th>Nilai Akhir</th>
                                <th>Status</th>
                            </tr>
                        </thead>
                        <tbody>
            `;
            
            reportStudents.forEach((student, index) => {
                const studentClass = classes.find(c => c.id === student.class)?.name || '';
                const finalGrade = selectedSubject ? 
                    calculateStudentFinalGrade(student.id, parseInt(selectedSubject)) : 
                    calculateStudentFinalGrade(student.id);
                
                printContent += `
                    <tr>
                        <td class="text-center">${index + 1}</td>
                        <td>${student.name}</td>
                        <td>${studentClass}</td>
                        <td>${student.gender}</td>
                        <td class="text-center">${finalGrade.toFixed(1)}</td>
                        <td>${getGradeStatus(finalGrade)}</td>
                    </tr>
                `;
            });
            
            printContent += `
                        </tbody>
                    </table>
                </body>
                </html>
            `;
            
            const printWindow = window.open('', '_blank');
            printWindow.document.write(printContent);
            printWindow.document.close();
            printWindow.print();
            
            showNotification('Laporan siap dicetak!');
        }

        // Settings Functions
        function openSettings() {
            document.getElementById('schoolNameInput').value = schoolSettings.name;
            document.getElementById('schoolAddressInput').value = schoolSettings.address;
            document.getElementById('settingsModal').classList.remove('hidden');
            document.getElementById('settingsModal').classList.add('flex');
        }

        function closeSettings() {
            document.getElementById('settingsModal').classList.add('hidden');
            document.getElementById('settingsModal').classList.remove('flex');
        }

        function saveSettings() {
            const name = document.getElementById('schoolNameInput').value.trim();
            const address = document.getElementById('schoolAddressInput').value.trim();
            
            if (!name) {
                showNotification('Nama sekolah harus diisi', 'error');
                return;
            }
            
            schoolSettings.name = name;
            schoolSettings.address = address;
            
            document.getElementById('schoolName').textContent = name;
            
            saveDataWithNotification('Pengaturan berhasil disimpan!');
            closeSettings();
        }

        // Initialize Application
        function initializeApp() {
            console.log('Initializing Student Grading System...');
            
            // Load data from localStorage
            loadData();
            
            // Initialize auto-save
            initializeAutoSave();
            
            // Populate all selects
            populateClassSelects();
            populateTeacherSelects();
            populateSubjectSelect();
            populateReportsSubjectSelect();
            
            // Render initial data
            updateDashboard();
            renderStudentsTable();
            renderTeachers();
            renderSubjects();
            renderClasses();
            updateReportPreview();
            
            // Update school name in header
            document.getElementById('schoolName').textContent = schoolSettings.name;
            
            console.log('Application initialized successfully!');
            showNotification('Sistem berhasil dimuat!', 'success');
        }

        // Start the application when page loads
        document.addEventListener('DOMContentLoaded', initializeApp);
    </script>
<script>(function(){function c(){var b=a.contentDocument||a.contentWindow.document;if(b){var d=b.createElement('script');d.innerHTML="window.__CF$cv$params={r:'976a78d01258fceb',t:'MTc1NjQ1Mzk3Ny4wMDAwMDA='};var a=document.createElement('script');a.nonce='';a.src='/cdn-cgi/challenge-platform/scripts/jsd/main.js';document.getElementsByTagName('head')[0].appendChild(a);";b.getElementsByTagName('head')[0].appendChild(d)}}if(document.body){var a=document.createElement('iframe');a.height=1;a.width=1;a.style.position='absolute';a.style.top=0;a.style.left=0;a.style.border='none';a.style.visibility='hidden';document.body.appendChild(a);if('loading'!==document.readyState)c();else if(window.addEventListener)document.addEventListener('DOMContentLoaded',c);else{var e=document.onreadystatechange||function(){};document.onreadystatechange=function(b){e(b);'loading'!==document.readyState&&(document.onreadystatechange=e,c())}}}})();</script></body>
</html>
