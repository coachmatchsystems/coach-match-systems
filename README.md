<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Coach Match Systems - Connecting Expertise with Ambition</title>
    <!-- Load Tailwind CSS for beautiful styling -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- Load Lucide Icons for nice visual elements -->
    <script src="https://unpkg.com/lucide@latest"></script>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;600;700;800&display=swap" rel="stylesheet">
    <style>
        /* Custom styles for the Inter font and smooth scrolling */
        body {
            font-family: 'Inter', sans-serif;
            scroll-behavior: smooth;
            background-color: #f7f9fb;
        }
        /* Style for the fixed message box (used instead of alert()) */
        #message-box {
            position: fixed;
            top: 1rem;
            right: 1rem;
            z-index: 1000;
            display: none;
            padding: 1rem 1.5rem;
            border-radius: 0.5rem;
            box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1), 0 2px 4px -2px rgba(0, 0, 0, 0.1);
        }
    </style>
</head>
<body>

    <!-- Notification/Message Box -->
    <div id="message-box" class="bg-green-500 text-white"></div>

    <!-- Firebase Imports and Setup (All JavaScript is contained here) -->
    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getAuth, signInAnonymously, signInWithCustomToken } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import { getFirestore, collection, addDoc, setLogLevel } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";
        
        // Use a debug log level to help with troubleshooting
        setLogLevel('Debug');

        // --- Global Variables (Provided by Canvas Environment) ---
        // Your unique application identifier
        const appId = typeof __app_id !== 'undefined' ? __app_id : 'default-app-id';
        const firebaseConfig = JSON.parse(typeof __firebase_config !== 'undefined' ? __firebase_config : '{}');
        const initialAuthToken = typeof __initial_auth_token !== 'undefined' ? __initial_auth_token : null;
        
        // --- Firebase Initialization ---
        let db;
        let auth;
        let isAuthReady = false;

        async function initFirebase() {
            try {
                const app = initializeApp(firebaseConfig);
                db = getFirestore(app);
                auth = getAuth(app);

                // Authenticate user (necessary to write data to Firestore)
                if (initialAuthToken) {
                    await signInWithCustomToken(auth, initialAuthToken);
                } else {
                    await signInAnonymously(auth);
                }
                isAuthReady = true;
                console.log("Firebase initialized and user authenticated.");
            } catch (error) {
                console.error("Error initializing Firebase:", error);
                isAuthReady = false;
            }
        }

        // --- Message Display Function ---
        // Displays a temporary message on the screen instead of using alert()
        window.showMessage = function(message, type = 'success') {
            const msgBox = document.getElementById('message-box');
            msgBox.textContent = message;

            msgBox.classList.remove('bg-green-500', 'bg-red-500');
            if (type === 'success') {
                msgBox.classList.add('bg-green-500');
            } else if (type === 'error') {
                msgBox.classList.add('bg-red-500');
            }

            msgBox.style.display = 'block';
            
            // Hide after 3 seconds
            setTimeout(() => {
                msgBox.style.display = 'none';
            }, 3000);
        }

        // --- Form Submission Handler (The core saving logic) ---
        window.handleFormSubmission = async function(event) {
            event.preventDefault(); // Stop the form from causing a page refresh
            
            const form = event.target;
            const role = form.querySelector('input[name="role"]').value;
            const email = form.querySelector('input[name="email"]').value;
            
            // Basic validation check
            if (!email || !email.includes('@')) {
                window.showMessage('Please enter a valid email address.', 'error');
                return;
            }

            // Check if Firebase is ready before sending data
            if (!isAuthReady || !db) {
                window.showMessage('System is still connecting. Please try again in a moment.', 'error');
                return;
            }

            try {
                // Defines the secure, public path for saving the waitlist emails
                const collectionPath = `/artifacts/${appId}/public/data/waitlist_leads`;
                
                // Saves the data to Firestore
                await addDoc(collection(db, collectionPath), {
                    email: email,
                    role: role,
                    timestamp: new Date().toISOString(),
                    userId: auth.currentUser?.uid || 'anonymous'
                });

                console.log(`Lead saved to Firestore: Role=${role}, Email=${email}`);
                window.showMessage(`Success! Your interest as a ${role} has been recorded.`);
                form.reset(); // Clear the form
            } catch (e) {
                console.error("Error adding document to Firestore: ", e);
                window.showMessage('Error saving lead. Please try again later.', 'error');
            }
        }

        // --- Initialization and Event Listeners ---
        document.addEventListener('DOMContentLoaded', () => {
            initFirebase();
            // Connects the Coach and Client forms to the saving function
            document.getElementById('coach-form').addEventListener('submit', window.handleFormSubmission);
            document.getElementById('client-form').addEventListener('submit', window.handleFormSubmission);
        });

        // Disable right-click for a better user experience
        document.addEventListener('contextmenu', event => event.preventDefault());

    </script>

    <!-- Header & Navigation -->
    <header class="sticky top-0 z-50 bg-white shadow-md">
        <div class="max-w-7xl mx-auto py-4 px-4 sm:px-6 lg:px-8">
            <div class="flex justify-between items-center h-16">
                <!-- Logo -->
                <a href="#" class="flex items-center space-x-2">
                    <span class="text-2xl font-extrabold text-indigo-600 tracking-tight">Coach Match Systems</span>
                </a>
                <!-- Navigation Links -->
                <nav class="hidden md:block space-x-8">
                    <a href="#coaches" class="text-gray-600 hover:text-indigo-600 transition duration-150 ease-in-out font-medium">For Coaches</a>
                    <a href="#clients" class="text-gray-600 hover:text-indigo-600 transition duration-150 ease-in-out font-medium">For Clients</a>
                    <a href="#mission" class="text-gray-600 hover:text-indigo-600 transition duration-150 ease-in-out font-medium">Our Mission</a>
                </nav>
                <!-- CTA Button -->
                <a href="#clients" class="hidden md:inline-flex items-center px-4 py-2 border border-transparent text-sm font-medium rounded-lg shadow-sm text-white bg-indigo-600 hover:bg-indigo-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500 transition duration-150 ease-in-out">
                    Get Matched
                </a>
            </div>
        </div>
    </header>

    <!-- Hero Section -->
    <main>
        <div class="relative overflow-hidden pt-10 pb-20 sm:pt-16 lg:pt-24 bg-indigo-50">
            <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 text-center">
                <h1 class="text-4xl sm:text-5xl lg:text-6xl font-extrabold tracking-tight text-gray-900">
                    <span class="block">Connecting Expertise with Ambition.</span>
                    <span class="block text-indigo-600 mt-2">Find Your Perfect Match.</span>
                </h1>
                <p class="mt-4 max-w-3xl mx-auto text-xl text-gray-500">
                    Coach Match Systems is building the premier platform where skilled professionals meet individuals ready for transformation.
                </p>
                <div class="mt-10 flex flex-col sm:flex-row justify-center space-y-4 sm:space-y-0 sm:space-x-6">
                    <a href="#coaches" class="w-full sm:w-auto inline-flex items-center justify-center px-8 py-3 border border-transparent text-base font-medium rounded-lg shadow-lg text-white bg-indigo-600 hover:bg-indigo-700 transition duration-300">
                        I'm a Coach (Attract Clients)
                    </a>
                    <a href="#clients" class="w-full sm:w-auto inline-flex items-center justify-center px-8 py-3 border border-indigo-600 text-base font-medium rounded-lg text-indigo-700 bg-white hover:bg-indigo-50 transition duration-300 shadow-lg">
                        I need a Coach (Achieve Goals)
                    </a>
                </div>
            </div>
        </div>

        <!-- Value Proposition Section -->
        <div id="mission" class="py-16 bg-white">
            <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
                <div class="lg:text-center">
                    <h2 class="text-base text-indigo-600 font-semibold tracking-wide uppercase">Our Mission</h2>
                    <p class="mt-2 text-3xl leading-8 font-extrabold tracking-tight text-gray-900 sm:text-4xl">
                        A System Designed for Growth
                    </p>
                </div>

                <div class="mt-10">
                    <dl class="space-y-10 md:space-y-0 md:grid md:grid-cols-3 md:gap-x-8 md:gap-y-10">
                        <!-- Feature 1 -->
                        <div class="relative bg-gray-50 p-6 rounded-xl shadow-lg hover:shadow-xl transition duration-300">
                            <div class="flex items-center justify-center h-12 w-12 rounded-full bg-indigo-100 text-indigo-600">
                                <i data-lucide="zap" class="w-6 h-6"></i>
                            </div>
                            <dt class="mt-4 text-lg leading-6 font-medium text-gray-900">Precision Matching</dt>
                            <dd class="mt-2 text-base text-gray-500">
                                Our algorithm will connect clients with coaches based on specialty, experience, and style, ensuring high-quality compatibility from the start.
                            </dd>
                        </div>
                        <!-- Feature 2 -->
                        <div class="relative bg-gray-50 p-6 rounded-xl shadow-lg hover:shadow-xl transition duration-300">
                            <div class="flex items-center justify-center h-12 w-12 rounded-full bg-indigo-100 text-indigo-600">
                                <i data-lucide="trending-up" class="w-6 h-6"></i>
                            </div>
                            <dt class="mt-4 text-lg leading-6 font-medium text-gray-900">Streamlined Client Acquisition</dt>
                            <dd class="mt-2 text-base text-gray-500">
                                Coaches can focus solely on coaching. We handle the marketing, scheduling, and payment infrastructure to fill your calendar effortlessly.
                            </dd>
                        </div>
                        <!-- Feature 3 -->
                        <div class="relative bg-gray-50 p-6 rounded-xl shadow-lg hover:shadow-xl transition duration-300">
                            <div class="flex items-center justify-center h-12 w-12 rounded-full bg-indigo-100 text-indigo-600">
                                <i data-lucide="check-circle" class="w-6 h-6"></i>
                            </div>
                            <dt class="mt-4 text-lg leading-6 font-medium text-gray-900">Verified Quality & Trust</dt>
                            <dd class="mt-2 text-base text-gray-500">
                                Every coach profile will be thoroughly vetted. Clients can trust they are working with certified and highly rated professionals.
                            </dd>
                        </div>
                    </dl>
                </div>
            </div>
        </div>

        <!-- Section for Coaches (Lead Capture) -->
        <div id="coaches" class="bg-indigo-700 py-16 sm:py-24">
            <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
                <div class="lg:grid lg:grid-cols-12 lg:gap-8">
                    <div class="lg:col-span-6">
                        <h2 class="text-3xl font-extrabold text-white sm:text-4xl">
                            Attract Ideal Clients. Grow Your Practice.
                        </h2>
                        <p class="mt-4 text-lg text-indigo-200">
                            We're launching soon and seeking passionate, certified coaches in all specialties (life, career, executive, health, etc.). Join our priority waitlist to be among the first onboarded and receive launch-day benefits.
                        </p>
                        <ul class="mt-6 space-y-3 text-indigo-200 text-base">
                            <li class="flex items-center"><i data-lucide="briefcase" class="w-5 h-5 mr-2"></i> High-quality client leads delivered to you.</li>
                            <li class="flex items-center"><i data-lucide="calendar-check" class="w-5 h-5 mr-2"></i> Integrated scheduling and billing tools.</li>
                            <li class="flex items-center"><i data-lucide="shield-check" class="w-5 h-5 mr-2"></i> Build your brand with a trusted platform.</li>
                        </ul>
                    </div>
                    
                    <!-- Coach Sign-Up Form -->
                    <div class="mt-10 lg:mt-0 lg:col-span-6">
                        <div class="bg-white p-8 rounded-xl shadow-2xl">
                            <h3 class="text-2xl font-bold text-gray-900 mb-6">Join the Coach Waitlist</h3>
                            <form id="coach-form" class="space-y-4">
                                <input type="hidden" name="role" value="Coach">
                                <div>
                                    <label for="coach-email" class="block text-sm font-medium text-gray-700">Email Address</label>
                                    <div class="mt-1">
                                        <input id="coach-email" name="email" type="email" autocomplete="email" required class="appearance-none block w-full px-4 py-2 border border-gray-300 rounded-lg shadow-sm placeholder-gray-400 focus:outline-none focus:ring-indigo-500 focus:border-indigo-500 sm:text-sm">
                                    </div>
                                </div>
                                <button type="submit" class="w-full flex justify-center py-3 px-4 border border-transparent rounded-lg shadow-lg text-sm font-medium text-white bg-indigo-600 hover:bg-indigo-700 transition duration-300">
                                    Sign Up for Priority Access
                                </button>
                            </form>
                        </div>
                    </div>
                </div>
            </div>
        </div>

        <!-- Section for Clients (Lead Capture) -->
        <div id="clients" class="bg-white py-16 sm:py-24">
            <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
                <div class="lg:grid lg:grid-cols-12 lg:gap-8">
                     <!-- Client Sign-Up Form -->
                    <div class="order-2 lg:order-1 mt-10 lg:mt-0 lg:col-span-6">
                        <div class="bg-gray-50 p-8 rounded-xl shadow-2xl">
                            <h3 class="text-2xl font-bold text-gray-900 mb-6">Find Your Future Coach</h3>
                            <form id="client-form" class="space-y-4">
                                <input type="hidden" name="role" value="Client">
                                <div>
                                    <label for="client-email" class="block text-sm font-medium text-gray-700">Email Address</label>
                                    <div class="mt-1">
                                        <input id="client-email" name="email" type="email" autocomplete="email" required class="appearance-none block w-full px-4 py-2 border border-gray-300 rounded-lg shadow-sm placeholder-gray-400 focus:outline-none focus:ring-indigo-500 focus:border-indigo-500 sm:text-sm">
                                    </div>
                                </div>
                                <button type="submit" class="w-full flex justify-center py-3 px-4 border border-transparent rounded-lg shadow-lg text-sm font-medium text-white bg-indigo-600 hover:bg-indigo-700 transition duration-300">
                                    Notify Me When Ready
                                </button>
                            </form>
                        </div>
                    </div>

                    <div class="order-1 lg:order-2 lg:col-span-6 lg:ml-10">
                        <h2 class="text-3xl font-extrabold text-gray-900 sm:text-4xl">
                            Ready to Transform Your Life?
                        </h2>
                        <p class="mt-4 text-lg text-gray-600">
                            Whether you need guidance for a career change, personal development, or fitness goals, we're building a network of trusted experts to meet your exact needs.
                        </p>
                        <ul class="mt-6 space-y-3 text-gray-600 text-base">
                            <li class="flex items-center"><i data-lucide="search" class="w-5 h-5 mr-2 text-indigo-600"></i> Search by niche: executive, health, life, etc.</li>
                            <li class="flex items-center"><i data-lucide="star" class="w-5 h-5 mr-2 text-indigo-600"></i> View verified reviews and credentials.</li>
                            <li class="flex items-center"><i data-lucide="message-square" class="w-5 h-5 mr-2 text-indigo-600"></i> Seamlessly book an introductory session.</li>
                        </ul>
                    </div>
                </div>
            </div>
        </div>

    </main>

    <!-- Footer -->
    <footer class="bg-gray-800">
        <div class="max-w-7xl mx-auto py-8 px-4 sm:px-6 lg:px-8">
            <div class="flex flex-wrap justify-between items-center border-t border-gray-700 pt-6">
                <!-- Copyright -->
                <p class="text-base text-gray-400">&copy; 2025 <strong class="text-gray-300">Coach Match Systems</strong>. All rights reserved.</p>
                <div class="flex space-x-6">
                    <a href="#coaches" class="text-gray-400 hover:text-indigo-400 text-sm">Coaches</a>
                    <a href="#clients" class="text-gray-400 hover:text-indigo-400 text-sm">Clients</a>
                    <!-- Direct link to your preferred email address -->
                    <a href="mailto:coachmatchsystems@gmail.com" class="text-gray-400 hover:text-indigo-400 text-sm">Contact Us</a>
                    <!-- Link to the legal document you created -->
                    <a href="privacy_policy.md" class="text-gray-400 hover:text-indigo-400 text-sm">Privacy Policy</a>
                </div>
            </div>
        </div>
    </footer>

    <!-- Initialize Lucide icons on the page -->
    <script>
        lucide.createIcons();
    </script>
</body>
</html>
