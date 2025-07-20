<!DOCTYPE html>
<html lang="pl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Job&TradeApp - Ogłoszenia i Handel</title>
    
    <script src="https://cdn.tailwindcss.com"></script>
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet">
    <script src="https://unpkg.com/lucide@latest"></script>

    <style>
        body {
            font-family: 'Inter', sans-serif;
        }
        .brand-orange { color: #ff5a00; }
        .bg-brand-orange { background-color: #ff5a00; }
        
        /* Style dla nawigacji i stopki */
        .legacy-nav { background-color: #333; }
        .legacy-nav a:hover, .legacy-nav a.active { background-color: #555; }
        .legacy-footer { background-color: #444; }

        /* Style dla modala (okna dialogowego) */
        .modal-overlay {
            position: fixed;
            top: 0;
            left: 0;
            right: 0;
            bottom: 0;
            background-color: rgba(0, 0, 0, 0.7);
            display: flex;
            align-items: center;
            justify-content: center;
            z-index: 50;
            opacity: 0;
            pointer-events: none;
            transition: opacity 0.3s ease;
        }
        .modal-overlay.visible {
            opacity: 1;
            pointer-events: auto;
        }
        .modal-container {
            background-color: white;
            border-radius: 0.75rem;
            box-shadow: 0 25px 50px -12px rgba(0, 0, 0, 0.25);
            max-width: 42rem; /* 672px */
            width: 95%;
            max-height: 90vh;
            overflow-y: auto;
            transform: scale(0.95);
            transition: transform 0.3s ease;
        }
        .modal-overlay.visible .modal-container {
            transform: scale(1);
        }

        /* Style dla testu predyspozycji */
        .progress-bar-fill { transition: width 0.5s ease-in-out; }
        .question-card { transition: opacity 0.3s ease, transform 0.3s ease; }
        .question-card.hidden {
            opacity: 0;
            transform: translateX(-20px);
            position: absolute;
            pointer-events: none;
        }
        .option-button { transition: background-color 0.2s, border-color 0.2s; }
        .option-button.selected {
            background-color: #FEF3C7; /* Jasnożółty */
            border-color: #FBBF24; /* Ciemniejszy żółty */
        }
        .filter-link.active {
            background-color: #ff5a00;
            color: white;
        }
    </style>
</head>
<body class="bg-gray-100">

    <header class="bg-white p-4 border-b border-gray-200 sticky top-0 z-40 shadow-sm">
        <div class="container mx-auto flex justify-between items-center">
            <a href="#" class="text-3xl font-bold brand-orange" onclick="window.location.reload()">Job&TradeApp</a>
            <div class="hidden md:flex flex-grow max-w-lg mx-4">
                <input type="search" placeholder="Czego szukasz?" class="w-full px-4 py-2 border-2 border-gray-300 rounded-l-lg focus:outline-none focus:border-orange-500">
                <button class="bg-brand-orange text-white font-semibold px-6 rounded-r-lg hover:bg-opacity-90 transition">Szukaj</button>
            </div>
            <div id="user-actions-logged-out" class="flex items-center space-x-4">
                <a href="#" id="login-btn" class="text-gray-600 hover:brand-orange">Zaloguj się</a>
                <a href="#" id="register-btn" class="bg-brand-orange text-white font-semibold px-4 py-2 rounded-lg hover:bg-opacity-90 transition">Zarejestruj się</a>
            </div>
            <div id="user-actions-logged-in" class="hidden items-center space-x-4">
                 <!-- Panel użytkownika będzie wstrzykiwany tutaj -->
            </div>
        </div>
    </header>

    <nav class="legacy-nav py-2 shadow-md">
        <div class="container mx-auto flex justify-center flex-wrap">
            <a data-category="czesci-samochodowe" class="text-white px-4 py-2 cursor-pointer rounded">Części samochodowe</a>
            <a data-category="artykuly-spozywcze" class="text-white px-4 py-2 cursor-pointer rounded">Artykuły spożywcze</a>
            <a data-category="alkohol" class="text-white px-4 py-2 cursor-pointer rounded">Alkohol</a>
            <a data-category="oferty-pracy" id="job-offers-link" class="text-white px-4 py-2 cursor-pointer rounded">Oferty pracy za granicą</a>
            <a data-category="cyberbezpieczenstwo" class="text-white px-4 py-2 cursor-pointer rounded">Cyberbezpieczeństwo</a>
        </div>
    </nav>

    <main class="container mx-auto py-8" id="main-content">
        <h2 class="text-3xl font-bold text-gray-800 mb-6" id="page-title">Wybierz kategorię, aby zobaczyć oferty</h2>
        <div id="content-container"></div>
    </main>

    <footer class="legacy-footer text-center py-8 mt-8">
        <div class="container mx-auto text-gray-400">
            <p>
                <a href="#" class="text-gray-300 hover:text-white mx-2">Regulamin</a> | 
                <a href="#" class="text-gray-300 hover:text-white mx-2">Polityka prywatności</a> | 
                <a href="#" class="text-gray-300 hover:text-white mx-2">Pomoc</a> | 
                <a href="#" class="text-gray-300 hover:text-white mx-2">Kontakt</a>
            </p>
            <p class="mt-4 text-sm">&copy; 2025 Job&TradeApp. Wszelkie prawa zastrzeżone.</p>
        </div>
    </footer>

    <!-- MODAL CONTAINER -->
    <div id="modal-overlay" class="modal-overlay">
        <div id="modal-container" class="modal-container">
            <!-- Zawartość modala będzie wstrzykiwana tutaj przez JavaScript -->
        </div>
    </div>

    <script>
    document.addEventListener('DOMContentLoaded', () => {
        // --- SYMULACJA BAZY DANYCH I STANU UŻYTKOWNIKA ---
        const db = {
            sellers: {
                'auto-czesci-krakow': { name: 'AutoCzesci-Krakow', rating: 4.9, image: 'https://placehold.co/100x100/333/fff?text=A-K' },
                'moto-sklep': { name: 'Moto-Sklep', rating: 4.8, image: 'https://placehold.co/100x100/555/fff?text=M-S' },
                'garaz-majstra': { name: 'Garaż Majstra', rating: 4.6, image: 'https://placehold.co/100x100/777/fff?text=G-M' },
                'piekarnia-zloty-klos': { name: 'Piekarnia Złoty Kłos', rating: 4.9, image: 'https://placehold.co/100x100/c68a3e/fff?text=Piek' },
                'lokalna-mleczarnia': { name: 'Lokalna Mleczarnia', rating: 4.7, image: 'https://placehold.co/100x100/e6f2ff/000?text=Mlek' },
                'sadownik-jan': { name: 'Sadownik Jan', rating: 4.8, image: 'https://placehold.co/100x100/d92121/fff?text=Sad' },
                'browar-rzemieslnik': { name: 'Browar Rzemieślnik', rating: 4.9, image: 'https://placehold.co/100x100/f4b74a/000?text=BR' },
                'pinta': { name: 'PINTA', rating: 4.8, image: 'https://placehold.co/100x100/1a1a1a/fff?text=Pinta' },
                'lokalne-piwko': { name: 'Lokalne Piwko', rating: 4.5, image: 'https://placehold.co/100x100/3c2f2f/fff?text=LP' },
                'bau-meister-gmbh': { name: 'BauMeister GmbH', rating: 4.9, image: 'https://placehold.co/100x100/ffc107/000?text=Bau' },
                'care-pro-nl': { name: 'CarePro NL', rating: 4.8, image: 'https://placehold.co/100x100/28a745/fff?text=Care' },
                'cyber-secure': { name: 'CyberSecure Solutions', rating: 4.9, image: 'https://placehold.co/100x100/0d47a1/fff?text=CSS' },
                'secure-net': { name: 'SecureNet Experts', rating: 4.8, image: 'https://placehold.co/100x100/00695c/fff?text=SNE' },
                'data-protect': { name: 'DataProtect Inc.', rating: 4.7, image: 'https://placehold.co/100x100/4a148c/fff?text=DPI' }
            },
            'czesci-samochodowe': [
                { id: 1, name: 'Opony letnie Michelin Pilot Sport 4', price: '450,00 zł', sellerId: 'auto-czesci-krakow', image: 'https://placehold.co/400x400/333/fff?text=Opony', subCategory: 'Opony i felgi', manufacturer: 'Michelin', condition: 'Nowe' },
                { id: 2, name: 'Klocki hamulcowe Brembo', price: '280,00 zł', sellerId: 'moto-sklep', image: 'https://placehold.co/400x400/555/fff?text=Klocki', subCategory: 'Układ hamulcowy', manufacturer: 'Brembo', condition: 'Nowe' },
                { id: 7, name: 'Olej silnikowy Castrol EDGE 5W-30', price: '150,00 zł', sellerId: 'auto-czesci-krakow', image: 'https://placehold.co/400x400/999/fff?text=Olej', subCategory: 'Filtry i oleje', manufacturer: 'Castrol', condition: 'Nowe' },
                { id: 8, name: 'Wycieraczki Bosch Aerotwin', price: '85,00 zł', sellerId: 'garaz-majstra', image: 'https://placehold.co/400x400/111/fff?text=Wycieraczki', subCategory: 'Części karoserii', manufacturer: 'Bosch', condition: 'Nowe' },
                { id: 15, name: 'Używany alternator Valeo', price: '250,00 zł', sellerId: 'garaz-majstra', image: 'https://placehold.co/400x400/444/fff?text=Alternator', subCategory: 'Elektryka', manufacturer: 'Valeo', condition: 'Używane' }
            ],
            'artykuly-spozywcze': [
                { id: 3, name: 'Chleb wiejski na zakwasie', price: '8,50 zł', sellerId: 'piekarnia-zloty-klos', image: 'https://placehold.co/400x400/c68a3e/fff?text=Chleb' },
                { id: 9, name: 'Mleko świeże 3.2%', price: '3,20 zł', sellerId: 'lokalna-mleczarnia', image: 'https://placehold.co/400x400/e6f2ff/000?text=Mleko' },
                { id: 10, name: 'Jabłka polskie (1kg)', price: '4,00 zł', sellerId: 'sadownik-jan', image: 'https://placehold.co/400x400/d92121/fff?text=Jabłka' },
                { id: 11, name: 'Rogal maślany', price: '2,50 zł', sellerId: 'piekarnia-zloty-klos', image: 'https://placehold.co/400x400/eac985/000?text=Rogal' }
            ],
            'alkohol': [
                { id: 4, name: 'Piwo Rzemieślnicze IPA', price: '12,00 zł', sellerId: 'browar-rzemieslnik', image: 'https://placehold.co/400x400/f4b74a/000?text=Piwo+IPA' },
                { id: 12, name: 'Atak Chmielu - PINTA', price: '10,50 zł', sellerId: 'pinta', image: 'https://placehold.co/400x400/1a1a1a/fff?text=Atak' },
                { id: 13, name: 'Porter Bałtycki', price: '15,00 zł', sellerId: 'browar-rzemieslnik', image: 'https://placehold.co/400x400/2c1e12/fff?text=Porter' },
                { id: 14, name: 'Jasne Pełne Regionalne', price: '8,00 zł', sellerId: 'lokalne-piwko', image: 'https://placehold.co/400x400/f9d71c/000?text=Jasne' }
            ],
            'oferty-pracy': [
                { id: 5, name: 'Pracownik budowlany', price: '18 EUR/h', sellerId: 'bau-meister-gmbh', location: 'Berlin, Niemcy', description: 'Poszukujemy doświadczonych pracowników budowlanych do pracy przy wznoszeniu budynków mieszkalnych. Wymagane doświadczenie.' },
                { id: 6, name: 'Opiekunka osób starszych', price: '2500 EUR/miesiąc', sellerId: 'care-pro-nl', location: 'Amsterdam, Holandia', description: 'Praca z zamieszkaniem. Wymagana znajomość języka angielskiego lub niderlandzkiego w stopniu komunikatywnym.' }
            ],
            'cyberbezpieczenstwo': [
                { id: 16, name: 'Pentester (Specjalista ds. Testów Penetracyjnych)', price: '15 000 - 20 000 PLN', sellerId: 'cyber-secure', location: 'Warszawa (Zdalnie)', description: 'Poszukujemy doświadczonego pentestera do przeprowadzania testów bezpieczeństwa aplikacji webowych i mobilnych.' },
                { id: 17, name: 'Analityk Bezpieczeństwa IT', price: '12 000 - 16 000 PLN', sellerId: 'secure-net', location: 'Kraków', description: 'Analiza i reagowanie na incydenty bezpieczeństwa, monitorowanie systemów SIEM.' },
                { id: 18, name: 'Specjalista SOC', price: '10 000 - 14 000 PLN', sellerId: 'data-protect', location: 'Wrocław', description: 'Praca w Security Operations Center, analiza logów i alertów bezpieczeństwa.' },
                { id: 19, name: 'Konsultant ds. Cyberbezpieczeństwa', price: '18 000 - 25 000 PLN', sellerId: 'cyber-secure', location: 'Praca zdalna', description: 'Doradztwo w zakresie strategii bezpieczeństwa, audyty i wdrażanie standardów ISO 27001.' },
                { id: 20, name: 'Inżynier Bezpieczeństwa Sieciowego', price: '14 000 - 19 000 PLN', sellerId: 'secure-net', location: 'Gdańsk', description: 'Konfiguracja i zarządzanie urządzeniami sieciowymi (firewalle, IPS/IDS).' },
                { id: 21, name: 'Incident Responder', price: '16 000 - 22 000 PLN', sellerId: 'cyber-secure', location: 'Warszawa', description: 'Zarządzanie incydentami bezpieczeństwa, od wykrycia po usunięcie skutków ataku.' }
            ]
        };

        let state = {
            currentUser: null,
            currentCategory: null,
        };

        // --- ELEMENTY DOM ---
        const pageTitle = document.getElementById('page-title');
        const contentContainer = document.getElementById('content-container');
        const navLinks = document.querySelectorAll('nav a');
        const modalOverlay = document.getElementById('modal-overlay');
        const modalContainer = document.getElementById('modal-container');
        const loggedOutView = document.getElementById('user-actions-logged-out');
        const loggedInView = document.getElementById('user-actions-logged-in');

        // --- SZABLONY HTML (WIDOKI) ---
        const views = {
            loginChoice: `
                <div class="p-8 text-center">
                    <h2 class="text-3xl font-bold text-gray-800 mb-4">Witaj w Job&TradeApp!</h2>
                    <p class="text-gray-600 mb-8">Wybierz, w jakiej roli chcesz się zalogować.</p>
                    <div class="flex flex-col sm:flex-row gap-4">
                        <button data-type="employee" class="login-choice-btn flex-1 bg-blue-500 text-white p-6 rounded-lg text-left hover:bg-blue-600 transition">
                            <h3 class="text-xl font-bold">Szukam / Znajdź pracę</h3>
                            <p class="mt-1 text-blue-100">Przeglądaj oferty pracy i aplikuj.</p>
                        </button>
                        <button data-type="seller" class="login-choice-btn flex-1 bg-green-500 text-white p-6 rounded-lg text-left hover:bg-green-600 transition">
                            <h3 class="text-xl font-bold">Klient B2B</h3>
                            <p class="mt-1 text-green-100">Zarządzaj produktami i sprzedażą.</p>
                        </button>
                    </div>
                </div>
            `,
            register: `
                <div class="p-8">
                    <h2 class="text-3xl font-bold text-center text-gray-800 mb-2">Stwórz nowe konto</h2>
                    <p class="text-center text-gray-500 mb-8">Dołącz do nas i odkryj nowe możliwości!</p>
                    <form id="register-form">
                        <div class="mb-6">
                            <label class="block text-gray-700 text-sm font-bold mb-2">Typ konta:</label>
                            <div class="flex rounded-lg border border-gray-300 p-1 bg-gray-100">
                                <button type="button" data-type="employee" class="account-type-btn flex-1 p-2 rounded-md text-center font-medium bg-brand-orange text-white shadow">Pracownik</button>
                                <button type="button" data-type="seller" class="account-type-btn flex-1 p-2 rounded-md text-center font-medium text-gray-500">Sprzedawca B2B</button>
                            </div>
                            <input type="hidden" id="accountType" name="accountType" value="employee">
                        </div>
                        <div class="mb-4">
                            <label for="name" class="block text-gray-700 text-sm font-bold mb-2">Imię i nazwisko / Nazwa firmy:</label>
                            <input type="text" id="name" class="w-full px-4 py-2 border rounded-lg focus:outline-none focus:ring-2 focus:ring-orange-400" required>
                        </div>
                        <div class="mb-4">
                            <label for="email" class="block text-gray-700 text-sm font-bold mb-2">Adres e-mail:</label>
                            <input type="email" id="email" class="w-full px-4 py-2 border rounded-lg focus:outline-none focus:ring-2 focus:ring-orange-400" required>
                        </div>
                        <div class="mb-6">
                            <label for="password" class="block text-gray-700 text-sm font-bold mb-2">Hasło:</label>
                            <input type="password" id="password" class="w-full px-4 py-2 border rounded-lg focus:outline-none focus:ring-2 focus:ring-orange-400" required>
                        </div>
                        <button type="submit" class="w-full bg-brand-orange text-white font-bold py-3 px-4 rounded-lg hover:bg-opacity-90 transition-transform hover:scale-105">Zarejestruj się</button>
                    </form>
                    <p class="text-center text-sm text-gray-500 mt-6">Masz już konto? <a href="#" id="show-login" class="font-medium brand-orange hover:underline">Zaloguj się</a></p>
                </div>
            `,
            login: `
                <div class="p-8">
                    <h2 class="text-3xl font-bold text-center text-gray-800 mb-8">Witaj z powrotem!</h2>
                    <form id="login-form">
                         <div class="mb-4">
                            <label for="email-login" class="block text-gray-700 text-sm font-bold mb-2">Adres e-mail:</label>
                            <input type="email" id="email-login" value="test@example.com" class="w-full px-4 py-2 border rounded-lg focus:outline-none focus:ring-2 focus:ring-orange-400" required>
                        </div>
                        <div class="mb-6">
                            <label for="password-login" class="block text-gray-700 text-sm font-bold mb-2">Hasło:</label>
                            <input type="password" id="password-login" value="password" class="w-full px-4 py-2 border rounded-lg focus:outline-none focus:ring-2 focus:ring-orange-400" required>
                        </div>
                        <button type="submit" class="w-full bg-brand-orange text-white font-bold py-3 px-4 rounded-lg hover:bg-opacity-90 transition-transform hover:scale-105">Zaloguj się</button>
                    </form>
                    <p class="text-center text-sm text-gray-500 mt-6">Nie masz konta? <a href="#" id="show-register" class="font-medium brand-orange hover:underline">Stwórz je teraz</a></p>
                </div>
            `,
            personalityTest: `
                <div id="test-root">
                    <div class="p-6 border-b border-gray-200">
                        <h1 class="text-2xl font-bold text-gray-800 text-center">Test predyspozycji zawodowych</h1>
                        <p class="text-center text-gray-500 mt-1">Odpowiedz na kilka pytań, a my stworzymy Twój profil zawodowy.</p>
                    </div>
                    <div class="w-full bg-gray-200">
                        <div id="progressBar" class="bg-brand-orange h-2 progress-bar-fill" style="width: 0%;"></div>
                    </div>
                    <div id="testContainer" class="p-8 relative min-h-[350px] flex items-center">
                        <!-- Pytania będą wstrzykiwane tutaj -->
                    </div>
                    <div class="p-6 bg-gray-50 border-t border-gray-200 flex justify-end">
                        <button id="nextButton" class="bg-brand-orange text-white font-bold py-3 px-8 rounded-lg hover:bg-opacity-90 transition-colors">Dalej</button>
                    </div>
                </div>
            `,
            employeePanel: `
                <span class="text-gray-700 font-medium">Witaj, <span id="welcome-name"></span>!</span>
                <a href="#" class="text-sm text-gray-600 hover:brand-orange">Mój Profil</a>
                <a href="#" class="text-sm text-gray-600 hover:brand-orange">Zapisane Oferty</a>
                <a href="#" class="text-sm text-gray-600 hover:brand-orange">Moje Aplikacje</a>
                <a href="#" id="logout-btn" class="bg-gray-600 text-white font-semibold px-4 py-2 rounded-lg hover:bg-gray-700 transition">Wyloguj</a>
            `,
            sellerPanel: `
                <span class="text-gray-700 font-medium">Panel Sprzedawcy: <span id="welcome-name"></span></span>
                <a href="#" class="text-sm text-gray-600 hover:brand-orange">Moje Produkty</a>
                <a href="#" class="text-sm text-gray-600 hover:brand-orange">Zamówienia</a>
                <a href="#" class="text-sm text-gray-600 hover:brand-orange">Dodaj Produkt</a>
                <a href="#" id="logout-btn" class="bg-gray-600 text-white font-semibold px-4 py-2 rounded-lg hover:bg-gray-700 transition">Wyloguj</a>
            `
        };

        // --- LOGIKA APLIKACJI ---
        
        function updateUI() {
            if (state.currentUser) {
                loggedOutView.classList.add('hidden');
                loggedInView.classList.remove('hidden');
                loggedInView.classList.add('flex');
                if (state.currentUser.type === 'employee') {
                    loggedInView.innerHTML = views.employeePanel;
                } else {
                    loggedInView.innerHTML = views.sellerPanel;
                }
                document.getElementById('welcome-name').textContent = state.currentUser.name;
                document.getElementById('logout-btn').addEventListener('click', (e) => { e.preventDefault(); handleLogout(); });

            } else {
                loggedOutView.classList.remove('hidden');
                loggedInView.classList.add('hidden');
            }
        }

        function openModal(viewName, options = {}) {
            modalContainer.innerHTML = views[viewName];
            modalOverlay.classList.add('visible');
            attachModalListeners(viewName, options);
            if (viewName === 'personalityTest') {
                initPersonalityTest();
            }
        }

        function closeModal() {
            modalOverlay.classList.remove('visible');
        }

        function attachModalListeners(viewName, options) {
            if (viewName === 'loginChoice') {
                document.querySelectorAll('.login-choice-btn').forEach(btn => {
                    btn.addEventListener('click', () => {
                        openModal('login', { type: btn.dataset.type });
                    });
                });
            }
            
            const showLogin = document.getElementById('show-login');
            const showRegister = document.getElementById('show-register');
            if (showLogin) showLogin.addEventListener('click', (e) => { e.preventDefault(); openModal('loginChoice'); });
            if (showRegister) showRegister.addEventListener('click', (e) => { e.preventDefault(); openModal('register'); });

            if (viewName === 'register') {
                document.getElementById('register-form').addEventListener('submit', handleRegister);
                const accountTypeBtns = document.querySelectorAll('.account-type-btn');
                accountTypeBtns.forEach(btn => {
                    btn.addEventListener('click', () => {
                        accountTypeBtns.forEach(b => b.classList.remove('bg-brand-orange', 'text-white', 'shadow'));
                        btn.classList.add('bg-brand-orange', 'text-white', 'shadow');
                        document.getElementById('accountType').value = btn.dataset.type;
                    });
                });
            }

            if (viewName === 'login') {
                document.getElementById('login-form').addEventListener('submit', (e) => handleLogin(e, options.type));
            }
        }
        
        function handleRegister(e) {
            e.preventDefault();
            const name = document.getElementById('name').value;
            const type = document.getElementById('accountType').value;
            if (!name) return;
            state.currentUser = { name, type };
            updateUI();

            if (type === 'employee') {
                openModal('personalityTest');
            } else {
                closeModal();
            }
        }

        function handleLogin(e, type) {
            e.preventDefault();
            const name = type === 'employee' ? 'Jan Pracownik' : 'Firma B2B';
            state.currentUser = { name, type };
            updateUI();
            closeModal();
        }

        function handleLogout() {
            state.currentUser = null;
            updateUI();
        }

        function renderContent(categoryKey) {
            state.currentCategory = categoryKey;
            pageTitle.textContent = document.querySelector(`nav a[data-category="${categoryKey}"]`).textContent;
            contentContainer.innerHTML = ''; // Wyczyść kontener

            if (categoryKey === 'czesci-samochodowe') {
                renderCarPartsPage();
            } else {
                renderComplexPage(categoryKey);
            }
        }
        
        function renderComplexPage(categoryKey) {
            contentContainer.className = 'space-y-12';
            const items = db[categoryKey] || [];
            const sellerIds = [...new Set(items.map(p => p.sellerId))];
            const categorySellers = sellerIds.map(id => ({ id, ...db.sellers[id] })).filter(s => s.name);
            const isJobCategory = ['oferty-pracy', 'cyberbezpieczenstwo'].includes(categoryKey);

            // Sekcja Sprzedawców
            const sellersSection = document.createElement('section');
            const sellersTitle = document.createElement('h3');
            sellersTitle.className = 'text-2xl font-bold text-gray-800 mb-4 border-b pb-2';
            sellersTitle.textContent = isJobCategory ? 'Wyróżnione Firmy / Agencje' : 'Wyróżnieni Sprzedawcy';
            sellersSection.appendChild(sellersTitle);

            const sellersGrid = document.createElement('div');
            sellersGrid.className = 'grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-6';
            
            categorySellers.forEach(seller => {
                const isSuperSeller = seller.rating >= 4.8;
                const sellerCard = document.createElement('div');
                sellerCard.className = 'bg-white p-4 rounded-lg shadow-md flex items-center space-x-4 hover:shadow-lg transition-shadow';
                sellerCard.innerHTML = `
                    <img src="${seller.image}" alt="Logo ${seller.name}" class="w-16 h-16 rounded-full border-2 border-gray-200">
                    <div class="flex-grow">
                        <h4 class="font-bold text-lg text-gray-900">${seller.name}</h4>
                        <div class="flex items-center mt-1">
                             <svg class="w-5 h-5 text-yellow-400" fill="currentColor" viewBox="0 0 20 20"><path d="M9.049 2.927c.3-.921 1.603-.921 1.902 0l1.07 3.292a1 1 0 00.95.69h3.462c.969 0 1.371 1.24.588 1.81l-2.8 2.034a1 1 0 00-.364 1.118l1.07 3.292c.3.921-.755 1.688-1.54 1.118l-2.8-2.034a1 1 0 00-1.175 0l-2.8 2.034c-.784.57-1.838-.197-1.539-1.118l1.07-3.292a1 1 0 00-.364-1.118L2.98 8.72c-.783-.57-.38-1.81.588-1.81h3.461a1 1 0 00.951-.69l1.07-3.292z"></path></svg>
                             <span class="text-gray-600 text-sm font-semibold ml-1">${seller.rating}</span>
                        </div>
                    </div>
                    ${isSuperSeller ? `<div class="ml-auto"><span class="bg-yellow-400 text-yellow-900 text-xs font-bold px-2.5 py-1 rounded-full">Super Sprzedawca</span></div>` : ''}
                `;
                sellersGrid.appendChild(sellerCard);
            });
            sellersSection.appendChild(sellersGrid);
            contentContainer.appendChild(sellersSection);

            // Sekcja Produktów / Ofert
            const itemsSection = document.createElement('section');
            const itemsTitle = document.createElement('h3');
            itemsTitle.className = 'text-2xl font-bold text-gray-800 mb-4 border-b pb-2';
            itemsTitle.textContent = isJobCategory ? 'Wszystkie Oferty' : 'Wszystkie Produkty';
            itemsSection.appendChild(itemsTitle);

            const itemsGrid = document.createElement('div');
            itemsGrid.className = isJobCategory 
                ? 'grid grid-cols-1 gap-5' 
                : 'grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 lg:grid-cols-4 gap-5';
            
            items.forEach(item => {
                const card = document.createElement('article');
                if (isJobCategory) {
                    card.className = 'bg-white rounded-lg shadow-md overflow-hidden flex flex-col sm:flex-row hover:shadow-xl transition-shadow duration-300';
                    card.innerHTML = `
                        <div class="p-5 flex-grow">
                            <h3 class="text-xl font-bold text-gray-900">${item.name}</h3>
                            <p class="font-semibold text-gray-600 mt-1">${db.sellers[item.sellerId].name}</p>
                            <p class="text-sm text-gray-500 mt-1">${item.location}</p>
                            <p class="text-gray-700 mt-3 text-sm">${item.description}</p>
                        </div>
                        <div class="p-5 bg-gray-50 flex flex-col justify-center items-center sm:items-end border-t sm:border-t-0 sm:border-l border-gray-200">
                             <p class="text-lg font-bold brand-orange">${item.price}</p>
                             <button class="mt-3 bg-green-500 text-white font-bold py-2 px-4 rounded-lg hover:bg-green-600 transition w-full sm:w-auto">Aplikuj</button>
                        </div>
                    `;
                } else {
                    card.className = 'bg-white rounded-lg shadow-md overflow-hidden flex flex-col hover:shadow-xl transition-shadow duration-300';
                    card.innerHTML = `
                        <img src="${item.image || 'https://placehold.co/400x400/ccc/fff?text=Brak+zdjęcia'}" alt="Zdjęcie: ${item.name}" class="w-full h-48 object-cover">
                        <div class="p-4 flex flex-col flex-grow">
                            <h3 class="text-lg font-semibold text-gray-800 flex-grow">${item.name}</h3>
                            <p class="text-sm text-gray-500 mt-1">Sprzedawca: ${db.sellers[item.sellerId].name}</p>
                            <p class="text-xl font-bold text-gray-900 mt-2">${item.price}</p>
                            <button class="mt-4 w-full bg-teal-500 text-white font-bold py-2 px-4 rounded-lg hover:bg-teal-600 transition">Dodaj do koszyka</button>
                        </div>
                    `;
                }
                itemsGrid.appendChild(card);
            });
            itemsSection.appendChild(itemsGrid);
            contentContainer.appendChild(itemsSection);
        }
        
        function renderCarPartsPage() {
            contentContainer.innerHTML = `
                <div class="bg-white p-6 rounded-lg shadow-md mb-8">
                    <h3 class="text-xl font-bold text-gray-800 mb-4">Znajdź części do swojego pojazdu</h3>
                    <div class="grid grid-cols-1 md:grid-cols-4 gap-4">
                        <select class="w-full p-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-orange-400">
                            <option>Wybierz markę</option>
                            <option>Volkswagen</option>
                            <option>BMW</option>
                            <option>Audi</option>
                        </select>
                        <select class="w-full p-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-orange-400">
                            <option>Wybierz model</option>
                        </select>
                        <select class="w-full p-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-orange-400">
                            <option>Wybierz rocznik</option>
                        </select>
                        <button class="w-full bg-brand-orange text-white font-bold rounded-md hover:bg-opacity-90">Szukaj</button>
                    </div>
                </div>
                <div class="flex flex-col md:flex-row gap-8">
                    <aside class="w-full md:w-1/4">
                        <div id="filters-container" class="space-y-6"></div>
                    </aside>
                    <div class="w-full md:w-3/4" id="car-parts-content"></div>
                </div>
            `;

            const carPartsContent = document.getElementById('car-parts-content');
            const filtersContainer = document.getElementById('filters-container');

            // Renderowanie filtrów
            const subCategories = [...new Set(db['czesci-samochodowe'].map(p => p.subCategory))];
            const manufacturers = [...new Set(db['czesci-samochodowe'].map(p => p.manufacturer))];
            const conditions = [...new Set(db['czesci-samochodowe'].map(p => p.condition))];

            const createFilterSection = (title, items, filterKey) => {
                const links = items.map(item => `<a href="#" class="block text-gray-700 hover:brand-orange py-1" data-filter="${filterKey}" data-value="${item}">${item}</a>`).join('');
                return `
                    <div>
                        <h4 class="font-bold text-lg mb-2 border-b pb-1">${title}</h4>
                        <div class="space-y-1">${links}</div>
                    </div>
                `;
            };
            
            filtersContainer.innerHTML = `
                ${createFilterSection('Podkategorie', subCategories, 'subCategory')}
                ${createFilterSection('Producenci', manufacturers, 'manufacturer')}
                ${createFilterSection('Stan', conditions, 'condition')}
            `;

            // Renderowanie sprzedawców i produktów
            renderCarPartsSellersAndProducts(carPartsContent, {});

            // Dodanie obsługi kliknięć na filtry i sprzedawców
            contentContainer.addEventListener('click', (e) => {
                if (e.target.matches('[data-filter]')) {
                    e.preventDefault();
                    // Logika filtrowania produktów - do zaimplementowania
                    alert(`Filtrowanie po: ${e.target.dataset.filter} = ${e.target.dataset.value}`);
                }
                if (e.target.closest('[data-seller-id]')) {
                    e.preventDefault();
                    const sellerId = e.target.closest('[data-seller-id]').dataset.sellerId;
                    renderCarPartsSellersAndProducts(carPartsContent, { sellerId: sellerId });
                }
            });
        }

        function renderCarPartsSellersAndProducts(container, filters) {
             container.innerHTML = ''; // Wyczyść kontener przed renderowaniem
            const items = db['czesci-samochodowe'] || [];
            const sellerIds = [...new Set(items.map(p => p.sellerId))];
            const categorySellers = sellerIds.map(id => ({ id, ...db.sellers[id] })).filter(s => s.name);

            // Sekcja Sprzedawców
            const sellersSection = document.createElement('section');
            const sellersTitle = document.createElement('h3');
            sellersTitle.className = 'text-2xl font-bold text-gray-800 mb-4';
            sellersTitle.textContent = 'Wyróżnieni Sprzedawcy';
            sellersSection.appendChild(sellersTitle);

            const sellersGrid = document.createElement('div');
            sellersGrid.className = 'grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-6';
            
            categorySellers.forEach(seller => {
                const isSuperSeller = seller.rating >= 4.8;
                const sellerCard = document.createElement('a');
                sellerCard.href = '#';
                sellerCard.dataset.sellerId = seller.id;
                sellerCard.className = 'bg-white p-4 rounded-lg shadow-md flex items-center space-x-4 hover:shadow-lg transition-shadow hover:ring-2 hover:ring-orange-400';
                sellerCard.innerHTML = `
                    <img src="${seller.image}" alt="Logo ${seller.name}" class="w-16 h-16 rounded-full border-2 border-gray-200 pointer-events-none">
                    <div class="flex-grow pointer-events-none">
                        <h4 class="font-bold text-lg text-gray-900">${seller.name}</h4>
                        <div class="flex items-center mt-1">
                             <svg class="w-5 h-5 text-yellow-400" fill="currentColor" viewBox="0 0 20 20"><path d="M9.049 2.927c.3-.921 1.603-.921 1.902 0l1.07 3.292a1 1 0 00.95.69h3.462c.969 0 1.371 1.24.588 1.81l-2.8 2.034a1 1 0 00-.364 1.118l1.07 3.292c.3.921-.755 1.688-1.54 1.118l-2.8-2.034a1 1 0 00-1.175 0l-2.8 2.034c-.784.57-1.838-.197-1.539-1.118l1.07-3.292a1 1 0 00-.364-1.118L2.98 8.72c-.783-.57-.38-1.81.588-1.81h3.461a1 1 0 00.951-.69l1.07-3.292z"></path></svg>
                             <span class="text-gray-600 text-sm font-semibold ml-1">${seller.rating}</span>
                        </div>
                    </div>
                    ${isSuperSeller ? `<div class="ml-auto pointer-events-none"><span class="bg-yellow-400 text-yellow-900 text-xs font-bold px-2.5 py-1 rounded-full">Super Sprzedawca</span></div>` : ''}
                `;
                sellersGrid.appendChild(sellerCard);
            });
            sellersSection.appendChild(sellersGrid);
            container.appendChild(sellersSection);

            // Sekcja Produktów
            const productsSection = document.createElement('section');
            productsSection.className = "mt-12";
            const productsTitle = document.createElement('h3');
            productsTitle.className = 'text-2xl font-bold text-gray-800 mb-4';
            productsTitle.textContent = filters.sellerId ? `Produkty od ${db.sellers[filters.sellerId].name}` : 'Wszystkie Produkty';
            productsSection.appendChild(productsTitle);

            const productsGrid = document.createElement('div');
            productsGrid.className = 'grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-5';
            
            const filteredItems = items.filter(item => {
                return !filters.sellerId || item.sellerId === filters.sellerId;
            });

            filteredItems.forEach(item => {
                const card = document.createElement('article');
                card.className = 'bg-white rounded-lg shadow-md overflow-hidden flex flex-col hover:shadow-xl transition-shadow duration-300';
                card.innerHTML = `
                    <img src="${item.image || 'https://placehold.co/400x400/ccc/fff?text=Brak+zdjęcia'}" alt="Zdjęcie: ${item.name}" class="w-full h-48 object-cover">
                    <div class="p-4 flex flex-col flex-grow">
                        <h3 class="text-lg font-semibold text-gray-800 flex-grow">${item.name}</h3>
                        <p class="text-sm text-gray-500 mt-1">Sprzedawca: ${db.sellers[item.sellerId].name}</p>
                        <p class="text-xl font-bold text-gray-900 mt-2">${item.price}</p>
                        <button class="mt-4 w-full bg-teal-500 text-white font-bold py-2 px-4 rounded-lg hover:bg-teal-600 transition">Dodaj do koszyka</button>
                    </div>
                `;
                productsGrid.appendChild(card);
            });
            productsSection.appendChild(productsGrid);
            container.appendChild(productsSection);
        }


        // --- LOGIKA TESTU PREDYSPOZYCJI ---
        function initPersonalityTest() {
            const testContainer = document.getElementById('testContainer');
            const nextButton = document.getElementById('nextButton');
            const progressBar = document.getElementById('progressBar');
            
            const testQuestions = [
                { q: "Jak najlepiej opiszesz swój styl pracy?", o: ["Wolę pracować samodzielnie i w skupieniu.", "Lubię pracować w zespole i współpracować z innymi.", "Dobrze odnajduję się w obu sytuacjach."] },
                { q: "Wykonywanie powtarzalnych, prostych zadań przez dłuższy czas jest dla Ciebie:", o: ["OK, pozwala mi to się nie przemęczać myśleniem.", "Męczące, szybko zaczynam się nudzić.", "Akceptowalne, jeśli wiem, jaki jest cel tej pracy."] },
                { q: "Jak ważna jest dla Ciebie precyzja i dbałość o szczegóły?", type: 'range', o: ["Nieważna", "Bardzo ważna"] },
                { q: "Które z poniższych środowisk pracy preferujesz?", o: ["Spokojne i ciche (np. magazyn, montaż).", "Dynamiczne i głośne (np. hala produkcyjna).", "Nie ma to dla mnie znaczenia."] },
                { q: "Czy posiadasz doświadczenie w obsłudze maszyn lub narzędzi?", o: ["Tak, mam doświadczenie.", "Nie, ale szybko się uczę.", "Nie mam i wolę pracę bez maszyn."] }
            ];

            let currentQuestionIndex = -1;

            function showQuestion() {
                currentQuestionIndex++;
                
                const progress = ((currentQuestionIndex) / testQuestions.length) * 100;
                progressBar.style.width = `${progress}%`;

                if (currentQuestionIndex >= testQuestions.length) {
                    testContainer.innerHTML = `
                        <div class="w-full text-center p-8">
                            <div class="flex justify-center mb-4"><svg class="w-16 h-16 text-green-500" fill="none" stroke="currentColor" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 12l2 2 4-4m6 2a9 9 0 11-18 0 9 9 0 0118 0z"></path></svg></div>
                            <h2 class="text-2xl font-bold text-gray-800 mb-2">Dziękujemy!</h2>
                            <p class="text-lg text-gray-600">Twój profil zawodowy został wygenerowany. Na jego podstawie będziemy dopasowywać dla Ciebie najlepsze oferty pracy.</p>
                        </div>`;
                    nextButton.textContent = 'Zobacz oferty pracy';
                    nextButton.onclick = () => {
                        closeModal();
                        document.querySelector('a[data-category="oferty-pracy"]').click();
                    };
                    return;
                }
                
                const qData = testQuestions[currentQuestionIndex];
                let optionsHtml = '';
                if (qData.type === 'range') {
                    optionsHtml = `
                        <div class="flex justify-between text-sm text-gray-500 mb-2"><span>${qData.o[0]}</span><span>${qData.o[1]}</span></div>
                        <input type="range" min="1" max="5" value="3" class="w-full h-2 bg-gray-200 rounded-lg appearance-none cursor-pointer accent-orange-500">
                    `;
                } else {
                    optionsHtml = qData.o.map(opt => `<button class="option-button w-full text-left p-4 border-2 border-gray-200 rounded-lg">${opt}</button>`).join('');
                }

                testContainer.innerHTML = `
                    <div class="question-card w-full">
                        <h2 class="text-xl font-semibold text-gray-800 mb-1">Pytanie ${currentQuestionIndex + 1} z ${testQuestions.length}</h2>
                        <p class="text-lg text-gray-700 mb-6">${qData.q}</p>
                        <div class="space-y-3">${optionsHtml}</div>
                    </div>`;

                if (currentQuestionIndex === testQuestions.length - 1) {
                    nextButton.textContent = 'Zakończ test';
                }

                testContainer.querySelectorAll('.option-button').forEach(btn => {
                    btn.addEventListener('click', () => {
                        const parent = btn.parentElement;
                        parent.querySelectorAll('.option-button').forEach(b => b.classList.remove('selected', 'border-yellow-400', 'bg-yellow-100'));
                        btn.classList.add('selected', 'border-yellow-400', 'bg-yellow-100');
                        setTimeout(showQuestion, 300);
                    });
                });
            }
            
            nextButton.onclick = showQuestion;
            showQuestion();
        }

        // --- INICJALIZACJA ---
        document.getElementById('register-btn').addEventListener('click', (e) => { e.preventDefault(); openModal('register'); });
        document.getElementById('login-btn').addEventListener('click', (e) => { e.preventDefault(); openModal('loginChoice'); });
        modalOverlay.addEventListener('click', (e) => { if (e.target === modalOverlay) closeModal(); });

        navLinks.forEach(link => {
            link.addEventListener('click', (e) => {
                const categoryKey = e.target.dataset.category;
                navLinks.forEach(l => l.classList.remove('active', 'bg-gray-700'));
                e.target.classList.add('active', 'bg-gray-700');
                renderContent(categoryKey);
            });
        });

        // Domyślne ustawienia
        updateUI();
        if (navLinks.length > 0) {
            navLinks[0].click();
        }
    });
    </script>
</body>
</html>
