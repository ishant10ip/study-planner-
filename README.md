# study-planner- 
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>SSC CGL Study Planner Pro</title>
    <link rel="stylesheet" href="src/output.css">
</head>
<body class="min-h-screen flex flex-col items-center p-2 font-sans theme-default">
    <!-- Sign-In/Sign-Up Modal -->
    <div id="authModal" class="fixed inset-0 bg-gray-800 bg-opacity-75 flex items-center justify-center z-50">
        <div class="bg-white rounded-lg p-6 w-full max-w-md">
            <h2 id="authTitle" class="text-2xl font-bold mb-4">Sign In</h2>
            <div id="authMessage" class="text-red-500 mb-4 hidden"></div>
            <div class="mb-4">
                <label class="block text-sm font-medium text-gray-700">Username</label>
                <input id="usernameInput" type="text" placeholder="Enter username" class="w-full p-2 border rounded-md focus:ring-2 focus:ring-blue-500">
            </div>
            <div class="mb-4">
                <label class="block text-sm font-medium text-gray-700">Password</label>
                <input id="passwordInput" type="password" placeholder="Enter password" class="w-full p-2 border rounded-md focus:ring-2 focus:ring-blue-500">
            </div>
            <div class="flex space-x-2">
                <button id="signInBtn" onclick="handleSignIn()" class="w-full bg-blue-600 text-white p-2 rounded-md hover:bg-blue-700 transition">Sign In</button>
                <button id="signUpBtn" onclick="handleSignUp()" class="w-full bg-green-600 text-white p-2 rounded-md hover:bg-green-700 transition">Sign Up</button>
            </div>
            <button id="toggleAuthBtn" onclick="toggleAuthMode()" class="mt-4 text-blue-600 hover:underline">Don't have an account? Sign Up</button>
        </div>
    </div>

    <!-- Main App -->
    <div id="mainApp" class="hidden">
        <div class="fixed top-0 left-0 h-full w-16 bg-gray-800 text-white flex flex-col items-center py-4">
            <select id="themeSelect" onchange="changeTheme()" class="w-12 p-1 text-sm bg-gray-700 rounded-md text-white">
                <option value="default">Default</option>
                <option value="forest">Forest</option>
                <option value="ocean">Ocean</option>
                <option value="sunset">Sunset</option>
                <option value="mountain">Mountain</option>
                <option value="library">Library</option>
                <option value="coffee-shop">Coffee Shop</option>
            </select>
            <button onclick="logout()" class="mt-auto text-sm bg-red-600 p-2 rounded-md hover:bg-red-700 transition">Logout</button>
        </div>
        <div class="w-full max-w-4xl rounded-xl shadow-lg p-6 app-container ml-16">
            <div class="header rounded-t-lg p-3 text-white text-center">
                <h1 class="text-2xl font-bold">SSC CGL Study Planner Pro</h1>
                <p id="welcomeMessage" class="text-sm"></p>
            </div>

            <!-- Task Input Form -->
            <div class="my-4 bg-gray-100 p-4 rounded-lg">
                <h2 class="text-lg font-semibold mb-3">Add New Task</h2>
                <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
                    <div>
                        <label class="block text-sm font-medium text-gray-700">Subject</label>
                        <select id="subjectInput" class="w-full p-2 border rounded-md focus:ring-2 focus:ring-blue-500" onchange="updateTopicSuggestions()">
                            <option value="Quant">Quantitative Aptitude</option>
                            <option value="English">English</option>
                            <option value="Reasoning">Reasoning</option>
                            <option value="GK">General Knowledge</option>
                        </select>
                    </div>
                    <div>
                        <label class="block text-sm font-medium text-gray-700">Topic</label>
                        <input id="topicInput" type="text" list="topicSuggestions" placeholder="e.g., Algebra" class="w-full p-2 border rounded-md focus:ring-2 focus:ring-blue-500">
                        <datalist id="topicSuggestions"></datalist>
                    </div>
                    <div>
                        <label class="block text-sm font-medium text-gray-700">Task Description</label>
                        <input id="taskInput" type="text" placeholder="e.g., Solve 50 Questions" class="w-full p-2 border rounded-md focus:ring-2 focus:ring-blue-500">
                    </div>
                    <div>
                        <label class="block text-sm font-medium text-gray-700">Time</label>
                        <input id="timeInput" type="time" class="w-full p-2 border rounded-md focus:ring-2 focus:ring-blue-500">
                    </div>
                    <div>
                        <label class="block text-sm font-medium text-gray-700">Date</label>
                        <input id="dateInput" type="date" class="w-full p-2 border rounded-md focus:ring-2 focus:ring-blue-500">
                    </div>
                    <div>
                        <label class="block text-sm font-medium text-gray-700">Duration (hours)</label>
                        <input id="durationInput" type="number" min="0.5" step="0.5" placeholder="e.g., 2" class="w-full p-2 border rounded-md focus:ring-2 focus:ring-blue-500">
                    </div>
                </div>
                <div class="mt-4 flex space-x-2">
                    <button onclick="addTask()" class="w-full text-white p-2 rounded-md add-task-btn transition">Add Task</button>
                    <button onclick="startPomodoro()" class="w-full bg-purple-500 text-white p-2 rounded-md hover:bg-purple-600 transition">Start Pomodoro (25 min)</button>
                </div>
                <span id="pomodoroTimer" class="text-gray-600 mt-2 block"></span>
            </div>

            <!-- Subject Tabs -->
            <div class="mb-4">
                <div class="flex space-x-2 border-b">
                    <button onclick="showSubject('Quant')" class="px-4 py-2 font-semibold text-blue-600 border-b-2 border-blue-600">Quant</button>
                    <button onclick="showSubject('English')" class="px-4 py-2 font-semibold text-green-600 hover:border-b-2 hover:border-green-600">English</button>
                    <button onclick="showSubject('Reasoning')" class="px-4 py-2 font-semibold text-yellow-600 hover:border-b-2 hover:border-yellow-600">Reasoning</button>
                    <button onclick="showSubject('GK')" class="px-4 py-2 font-semibold text-red-600 hover:border-b-2 hover:border-red-600">GK</button>
                </div>
            </div>

            <!-- Date Filter -->
            <div class="mb-4">
                <label class="block text-sm font-medium text-gray-700">View Tasks for:</label>
                <input id="filterDate" type="date" class="w-full md:w-1/3 p-2 border rounded-md focus:ring-2 focus:ring-blue-500" onchange="renderTimetable()">
            </div>

            <!-- Timetable Display -->
            <h2 class="text-lg font-semibold mb-2">Daily Timetable</h2>
            <div id="timetable" class="space-y-2 mb-6"></div>

            <!-- Study Records -->
            <h2 class="text-lg font-semibold mb-2">Study Records</h2>
            <div class="flex space-x-2 mb-4">
                <button onclick="showDailyRecords()" class="bg-gray-200 p-2 rounded-md hover:bg-gray-300 transition">Daily</button>
                <button onclick="showWeeklyRecords()" class="bg-gray-200 p-2 rounded-md hover:bg-gray-300 transition">Weekly</button>
                <button onclick="showMonthlyRecords()" class="bg-gray-200 p-2 rounded-md hover:bg-gray-300 transition">Monthly</button>
                <button onclick="downloadRecords()" class="text-white p-2 rounded-md download-btn transition">Download Records</button>
            </div>
            <div id="records" class="space-y-2"></div>

            <!-- My Activity -->
            <h2 class="text-lg font-semibold mb-2 mt-6">My Activity</h2>
            <div class="flex space-x-2 mb-4">
                <button onclick="showDailyActivity()" class="bg-gray-200 p-2 rounded-md hover:bg-gray-300 transition">Daily</button>
                <button onclick="showWeeklyActivity()" class="bg-gray-200 p-2 rounded-md hover:bg-gray-300 transition">Weekly</button>
                <button onclick="showMonthlyActivity()" class="bg-gray-200 p-2 rounded-md hover:bg-gray-300 transition">Monthly</button>
            </div>
            <div id="activity" class="space-y-2"></div>

            <!-- Revision Planner -->
            <h2 class="text-lg font-semibold mb-2 mt-6">Revision Suggestions</h2>
            <div id="revisionPlanner" class="space-y-2"></div>
        </div>
    </div>

    <!-- Audio for Alarm -->
    <audio id="alarmSound" src="https://www.soundjay.com/buttons/beep-01a.mp3" loop></audio>

    <script src="src/script.js"></script>
</body>
</html>
