<!DOCTYPE html>
<html lang="ar">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>📖 المصحف الإلكتروني</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.2/css/all.min.css">
    <style>
        :root {
            --primary-color: #2c3e50;
            --secondary-color: #4ca1af;
            --text-color: #333;
            --bg-color: #f8f9fa;
            --card-bg: #fff;
            --dark-bg: #1a1a2e;
            --dark-card: #16213e;
            --dark-text: #f8f9fa;
        }

        body {
            font-family: 'Amiri', serif;
            direction: rtl;
            background-color: var(--bg-color);
            color: var(--text-color);
            transition: all 0.3s ease;
            padding-bottom: 120px;
        }

        .dark-mode {
            background-color: var(--dark-bg);
            color: var(--dark-text);
        }

        .header {
            background: linear-gradient(135deg, var(--primary-color), var(--secondary-color));
            color: white;
            padding: 1.5rem;
            border-radius: 0 0 15px 15px;
            box-shadow: 0 4px 6px rgba(0,0,0,0.1);
            position: sticky;
            top: 0;
            z-index: 1000;
        }

        .surah-card {
            background: var(--card-bg);
            border-radius: 12px;
            padding: 1.5rem;
            margin-bottom: 1.5rem;
            box-shadow: 0 3px 10px rgba(0,0,0,0.1);
            transition: all 0.3s ease;
            position: relative;
            border: none;
        }

        .surah-card:hover {
            transform: translateY(-5px);
            box-shadow: 0 10px 20px rgba(0,0,0,0.15);
        }

        .audio-player {
            position: fixed;
            bottom: 0;
            left: 0;
            right: 0;
            background: var(--card-bg);
            padding: 1rem;
            box-shadow: 0 -2px 10px rgba(0,0,0,0.1);
            z-index: 1000;
            transition: all 0.3s ease;
        }

        .progress-container {
            width: 100%;
            height: 6px;
            background: #e9ecef;
            border-radius: 3px;
            margin: 10px 0;
            cursor: pointer;
        }

        .progress-bar {
            height: 100%;
            background: var(--secondary-color);
            border-radius: 3px;
            width: 0%;
            transition: width 0.1s linear;
        }

        .btn-primary {
            background-color: var(--secondary-color);
            border-color: var(--secondary-color);
        }

        .favorite-btn {
            position: absolute;
            top: 1rem;
            left: 1rem;
            background: transparent;
            border: none;
            font-size: 1.2rem;
            color: #ffc107;
            cursor: pointer;
        }

        @media (max-width: 768px) {
            .surah-card {
                padding: 1rem;
            }
            
            .header {
                padding: 1rem;
            }
        }
    </style>
</head>
<body>
    <header class="header">
        <div class="container">
            <div class="d-flex justify-content-between align-items-center">
                <h1 class="mb-0">📖 المصحف الإلكتروني</h1>
                <button id="theme-toggle" class="btn btn-light">
                    <i class="fas fa-moon"></i>
                </button>
            </div>
            
            <div class="mt-3">
                <input type="text" id="search" class="form-control" placeholder="🔍 ابحث عن سورة...">
            </div>
            
            <div class="mt-3">
                <select id="reciter-select" class="form-select">
                    <option value="afs">عبد الباسط عبد الصمد</option>
                    <option value="minsh">محمد صديق المنشاوي</option>
                    <option value="hus">مشاري العفاسي</option>
                    <option value="maher">ماهر المعيقلي</option>
                    <option value="sds">عبد الرحمن السديس</option>
                </select>
            </div>
        </div>
    </header>

    <div class="container mt-4">
        <div class="row" id="surah-list">
            <!-- سيتم تعبئته بالجافاسكريبت -->
        </div>
    </div>

    <div class="audio-player" style="display: none;">
        <div class="container">
            <div class="d-flex align-items-center mb-2">
                <div class="flex-grow-1">
                    <h6 id="current-surah-name" class="mb-0"></h6>
                    <small id="current-reciter-name" class="text-muted"></small>
                </div>
            </div>
            
            <audio id="quran-audio" controls class="w-100">
                <source id="audio-source" type="audio/mpeg">
            </audio>
            
            <div class="d-flex justify-content-between align-items-center mt-2">
                <button id="play-pause" class="btn btn-primary btn-sm">
                    <i class="fas fa-pause"></i>
                </button>
                
                <div class="progress-container flex-grow-1 mx-2">
                    <div class="progress-bar" id="progress-bar"></div>
                </div>
                
                <span id="current-time" class="text-nowrap">00:00</span>
            </div>
        </div>
    </div>

    <script>
        // بيانات التطبيق مع معالجة الأخطاء
        const app = {
            surahs: [],
            currentReciter: 'afs',
            favorites: JSON.parse(localStorage.getItem('favorites')) || [],
            reciters: {
                'afs': { name: 'عبد الباسط عبد الصمد', baseUrl: 'https://download.quranicaudio.com/quran/abdul_baset_murattal/' },
                'minsh': { name: 'محمد صديق المنشاوي', baseUrl: 'https://download.quranicaudio.com/quran/muhammad_siddeeq_al-minshaawee/' },
                'hus': { name: 'مشاري العفاسي', baseUrl: 'https://download.quranicaudio.com/quran/mishary_rashid_alafasy/' },
                'maher': { name: 'ماهر المعيقلي', baseUrl: 'https://download.quranicaudio.com/quran/maher_al_muaiqly/' },
                'sds': { name: 'عبد الرحمن السديس', baseUrl: 'https://download.quranicaudio.com/quran/abdurrahmaan_as-sudays/' }
            },
            currentSurah: null,
            audio: document.getElementById('quran-audio'),
            audioSource: document.getElementById('audio-source'),
            
            // تهيئة التطبيق
            async init() {
                try {
                    await this.loadSurahs();
                    this.setupEventListeners();
                    this.loadTheme();
                } catch (error) {
                    console.error('Error initializing app:', error);
                    this.showError('حدث خطأ في تحميل التطبيق');
                }
            },
            
            // جلب بيانات السور
            async loadSurahs() {
                try {
                    const response = await fetch('https://api.alquran.cloud/v1/surah');
                    
                    if (!response.ok) throw new Error('Network response was not ok');
                    
                    const data = await response.json();
                    
                    if (!data.data || !Array.isArray(data.data)) {
                        throw new Error('Invalid data format');
                    }
                    
                    this.surahs = data.data;
                    this.renderSurahs();
                } catch (error) {
                    console.error('Error loading surahs:', error);
                    this.showError('حدث خطأ في جلب بيانات السور');
                    
                    // بيانات احتياطية في حالة فشل API
                    this.surahs = [
                        {number: 1, name: "الفاتحة", englishNameTranslation: "الفاتحة", numberOfAyahs: 7},
                        {number: 2, name: "البقرة", englishNameTranslation: "البقرة", numberOfAyahs: 286}
                    ];
                    this.renderSurahs();
                }
            },
            
            // عرض السور
            renderSurahs() {
                const surahsToRender = this.surahs;
                elements.surahList.innerHTML = surahsToRender.map(surah => `
                    <div class="col-md-4 mb-4">
                        <div class="card surah-card h-100">
                            <button class="favorite-btn" onclick="app.toggleFavorite(${surah.number})">
                                <i class="fas fa-star ${this.isFavorite(surah.number) ? 'text-warning' : 'text-muted'}"></i>
                            </button>
                            
                            <div class="card-body">
                                <h3 class="card-title">${surah.name}</h3>
                                <p class="card-text text-muted">${surah.englishNameTranslation}</p>
                                
                                <div class="d-flex justify-content-between align-items-center mt-3">
                                    <span class="badge bg-light text-dark">
                                        ${surah.numberOfAyahs} آيات
                                    </span>
                                    
                                    <div>
                                        <button class="btn btn-sm btn-primary me-2" 
                                                onclick="app.playSurah(${surah.number}, '${surah.name}')">
                                            <i class="fas fa-play"></i> تشغيل
                                        </button>
                                        <button class="btn btn-sm btn-success" 
                                                onclick="app.downloadSurah(${surah.number}, '${surah.name}')">
                                            <i class="fas fa-download"></i> تحميل
                                        </button>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </div>
                `).join('');
            },
            
            // تشغيل السورة
            async playSurah(number, name) {
                try {
                    if (!number || !name) {
                        throw new Error('Invalid surah data');
                    }
                    
                    const reciter = this.reciters[this.currentReciter];
                    if (!reciter) {
                        throw new Error('Reciter not found');
                    }
                    
                    this.currentSurah = { number, name };
                    const audioUrl = `${reciter.baseUrl}${number.toString().padStart(3, '0')}.mp3`;
                    
                    // اختبار وجود الملف الصوتي
                    const audioExists = await this.checkAudioExists(audioUrl);
                    if (!audioExists) {
                        throw new Error('Audio file not found');
                    }
                    
                    this.audioSource.src = audioUrl;
                    this.audio.load();
                    
                    await this.audio.play().catch(e => {
                        throw new Error('Playback failed');
                    });
                    
                    this.updatePlayerUI(name, reciter.name);
                    
                } catch (error) {
                    console.error('Error playing surah:', error);
                    this.showError('تعذر تشغيل السورة، يرجى المحاولة لاحقاً');
                }
            },
            
            // تحميل السورة
            async downloadSurah(number, name) {
                try {
                    const reciter = this.reciters[this.currentReciter];
                    if (!reciter) {
                        throw new Error('Reciter not found');
                    }
                    
                    const audioUrl = `${reciter.baseUrl}${number.toString().padStart(3, '0')}.mp3`;
                    
                    // اختبار وجود الملف الصوتي
                    const audioExists = await this.checkAudioExists(audioUrl);
                    if (!audioExists) {
                        throw new Error('Audio file not found');
                    }
                    
                    // إنشاء رابط تحميل
                    const a = document.createElement('a');
                    a.href = audioUrl;
                    a.download = `سورة ${name} - ${reciter.name}.mp3`;
                    document.body.appendChild(a);
                    a.click();
                    document.body.removeChild(a);
                    
                } catch (error) {
                    console.error('Error downloading surah:', error);
                    this.showError('تعذر تحميل السورة، يرجى المحاولة لاحقاً');
                }
            },
            
            // التحقق من وجود الملف الصوتي
            async checkAudioExists(url) {
                try {
                    const response = await fetch(url, { method: 'HEAD' });
                    return response.ok;
                } catch {
                    return false;
                }
            },
            
            // تحديث واجهة المشغل
            updatePlayerUI(surahName, reciterName) {
                elements.playerContainer.style.display = 'block';
                elements.currentSurahName.textContent = surahName;
                elements.currentReciterName.textContent = reciterName;
            },
            
            // إدارة المفضلة
            toggleFavorite(number) {
                const index = this.favorites.indexOf(number);
                if (index === -1) {
                    this.favorites.push(number);
                } else {
                    this.favorites.splice(index, 1);
                }
                localStorage.setItem('favorites', JSON.stringify(this.favorites));
                this.renderSurahs();
            },
            
            isFavorite(number) {
                return this.favorites.includes(number);
            },
            
            // إدارة الوضع المظلم
            toggleTheme() {
                document.body.classList.toggle('dark-mode');
                localStorage.setItem('darkMode', document.body.classList.contains('dark-mode'));
                this.updateThemeIcon();
            },
            
            loadTheme() {
                if (localStorage.getItem('darkMode') === 'true') {
                    document.body.classList.add('dark-mode');
                }
                this.updateThemeIcon();
            },
            
            updateThemeIcon() {
                const icon = elements.themeToggle.querySelector('i');
                if (document.body.classList.contains('dark-mode')) {
                    icon.classList.replace('fa-moon', 'fa-sun');
                } else {
                    icon.classList.replace('fa-sun', 'fa-moon');
                }
            },
            
            // إعداد مستمعي الأحداث
            setupEventListeners() {
                // البحث
                elements.searchInput.addEventListener('input', (e) => {
                    const term = e.target.value.toLowerCase();
                    const filtered = this.surahs.filter(surah => 
                        surah.name.toLowerCase().includes(term) || 
                        surah.englishNameTranslation.toLowerCase().includes(term)
                    );
                    this.renderFilteredSurahs(filtered);
                });
                
                // تغيير القارئ
                elements.reciterSelect.addEventListener('change', (e) => {
                    this.currentReciter = e.target.value;
                    if (this.currentSurah) {
                        this.playSurah(this.currentSurah.number, this.currentSurah.name);
                    }
                });
                
                // التحكم في المشغل
                elements.playPauseBtn.addEventListener('click', () => {
                    this.togglePlayback();
                });
                
                // شريط التقدم
                this.audio.addEventListener('timeupdate', () => {
                    this.updateProgressBar();
                });
                
                document.querySelector('.progress-container').addEventListener('click', (e) => {
                    this.seekAudio(e);
                });
            },
            
            // عرض السور المفلترة
            renderFilteredSurahs(filteredSurahs) {
                elements.surahList.innerHTML = filteredSurahs.map(surah => `
                    <div class="col-md-4 mb-4">
                        <div class="card surah-card h-100">
                            <button class="favorite-btn" onclick="app.toggleFavorite(${surah.number})">
                                <i class="fas fa-star ${this.isFavorite(surah.number) ? 'text-warning' : 'text-muted'}"></i>
                            </button>
                            
                            <div class="card-body">
                                <h3 class="card-title">${surah.name}</h3>
                                <p class="card-text text-muted">${surah.englishNameTranslation}</p>
                                
                                <div class="d-flex justify-content-between align-items-center mt-3">
                                    <span class="badge bg-light text-dark">
                                        ${surah.numberOfAyahs} آيات
                                    </span>
                                    
                                    <div>
                                        <button class="btn btn-sm btn-primary me-2" 
                                                onclick="app.playSurah(${surah.number}, '${surah.name}')">
                                            <i class="fas fa-play"></i> تشغيل
                                        </button>
                                        <button class="btn btn-sm btn-success" 
                                                onclick="app.downloadSurah(${surah.number}, '${surah.name}')">
                                            <i class="fas fa-download"></i> تحميل
                                        </button>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </div>
                `).join('');
            },
            
            // التحكم في التشغيل
            togglePlayback() {
                if (this.audio.paused) {
                    this.audio.play();
                    elements.playPauseBtn.innerHTML = '<i class="fas fa-pause"></i>';
                } else {
                    this.audio.pause();
                    elements.playPauseBtn.innerHTML = '<i class="fas fa-play"></i>';
                }
            },
            
            // تحديث شريط التقدم
            updateProgressBar() {
                const progress = (this.audio.currentTime / this.audio.duration) * 100;
                elements.progressBar.style.width = `${progress}%`;
                
                const minutes = Math.floor(this.audio.currentTime / 60);
                const seconds = Math.floor(this.audio.currentTime % 60);
                elements.currentTime.textContent = `${minutes}:${seconds < 10 ? '0' : ''}${seconds}`;
            },
            
            // الانتقال إلى وقت معين في الصوت
            seekAudio(e) {
                const percent = e.offsetX / e.target.offsetWidth;
                this.audio.currentTime = percent * this.audio.duration;
            },
            
            // عرض الأخطاء
            showError(message) {
                const alert = document.createElement('div');
                alert.className = 'alert alert-danger mt-3';
                alert.textContent = message;
                elements.surahList.prepend(alert);
            }
        };

        // تعريف العناصر
        const elements = {
            surahList: document.getElementById('surah-list'),
            searchInput: document.getElementById('search'),
            themeToggle: document.getElementById('theme-toggle'),
            reciterSelect: document.getElementById('reciter-select'),
            playerContainer: document.querySelector('.audio-player'),
            playPauseBtn: document.getElementById('play-pause'),
            progressBar: document.getElementById('progress-bar'),
            currentTime: document.getElementById('current-time'),
            currentSurahName: document.getElementById('current-surah-name'),
            currentReciterName: document.getElementById('current-reciter-name')
        };

        // بدء التطبيق
        document.addEventListener('DOMContentLoaded', () => {
            app.init();
        });
    </script>
</body>
</html>
