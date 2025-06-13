# study-planner- 
// Request notification permission
if (Notification.permission !== "granted") {
    Notification.requestPermission();
}

// Initialize user state
let currentUser = null;
let users = JSON.parse(localStorage.getItem("users")) || {};
let tasks = [];
let studyRecords = { Quant: [], English: [], Reasoning: [], GK: [] };
let currentSubject = "Quant";
let currentTheme = localStorage.getItem("theme") || "default";

// Check if a user is already logged in
const loggedInUser = localStorage.getItem("currentUser");
if (loggedInUser) {
    currentUser = loggedInUser;
    loadUserData();
    showMainApp();
} else {
    document.getElementById("authModal").classList.remove("hidden");
}

document.body.className = `min-h-screen flex flex-col items-center p-2 font-sans theme-${currentTheme}`;
document.getElementById("themeSelect").value = currentTheme;

// Topic suggestions
const topicSuggestions = {
    Quant: ["Algebra", "Geometry", "Trigonometry", "Number System", "Data Interpretation"],
    English: ["Vocabulary", "Grammar", "Reading Comprehension", "Sentence Correction"],
    Reasoning: ["Coding-Decoding", "Analogies", "Series", "Puzzles"],
    GK: ["Current Affairs", "History", "Geography", "Polity", "Science"]
};

// Display current date and time in IST
function displayDateTime() {
    const options = {
        timeZone: "Asia/Kolkata",
        weekday: "long",
        year: "numeric",
        month: "long",
        day: "numeric",
        hour: "2-digit",
        minute: "2-digit",
        hour12: true
    };
    const now = new Date().toLocaleString("en-US", options);
    document.getElementById("currentDateTime").textContent = now;
}

// Update date and time every minute
setInterval(displayDateTime, 60000);

// Toggle between sign-in and sign-up modes
function toggleAuthMode() {
    const isSignIn = document.getElementById("authTitle").textContent === "Sign In";
    document.getElementById("authTitle").textContent = isSignIn ? "Sign Up" : "Sign In";
    document.getElementById("toggleAuthBtn").textContent = isSignIn ? "Already have an account? Sign In" : "Don't have an account? Sign Up";
    document.getElementById("signInBtn").classList.toggle("hidden", !isSignIn);
    document.getElementById("signUpBtn").classList.toggle("hidden", isSignIn);
    document.getElementById("authMessage").classList.add("hidden");
}

// Handle sign-in
function handleSignIn() {
    const username = document.getElementById("usernameInput").value;
    const password = document.getElementById("passwordInput").value;
    const authMessage = document.getElementById("authMessage");

    if (!username || !password) {
        authMessage.textContent = "Please fill all fields!";
        authMessage.classList.remove("hidden");
        return;
    }

    if (users[username] && users[username].password === password) {
        currentUser = username;
        localStorage.setItem("currentUser", currentUser);
        loadUserData();
        showMainApp();
    } else {
        authMessage.textContent = "Invalid username or password!";
        authMessage.classList.remove("hidden");
    }
}

// Handle sign-up
function handleSignUp() {
    const username = document.getElementById("usernameInput").value;
    const password = document.getElementById("passwordInput").value;
    const authMessage = document.getElementById("authMessage");

    if (!username || !password) {
        authMessage.textContent = "Please fill all fields!";
        authMessage.classList.remove("hidden");
        return;
    }

    if (users[username]) {
        authMessage.textContent = "Username already exists!";
        authMessage.classList.remove("hidden");
        return;
    }

    users[username] = { password, tasks: [], studyRecords: { Quant: [], English: [], Reasoning: [], GK: [] } };
    localStorage.setItem("users", JSON.stringify(users));
    currentUser = username;
    localStorage.setItem("currentUser", currentUser);
    loadUserData();
    showMainApp();
}

// Load user-specific data
function loadUserData() {
    const userData = users[currentUser];
    tasks = userData.tasks || [];
    studyRecords = userData.studyRecords || { Quant: [], English: [], Reasoning: [], GK: [] };
    document.getElementById("welcomeMessage").textContent = `Welcome, ${currentUser}!`;
    displayDateTime(); // Initial call to display date and time
}

// Save user-specific data
function saveUserData() {
    users[currentUser].tasks = tasks;
    users[currentUser].studyRecords = studyRecords;
    localStorage.setItem("users", JSON.stringify(users));
}

// Show main app after login
function showMainApp() {
    document.getElementById("authModal").classList.add("hidden");
    document.getElementById("mainApp").classList.remove("hidden");
    document.getElementById("filterDate").value = new Date().toISOString().slice(0, 10);
    updateTopicSuggestions();
    showSubject("Quant");
}

// Logout
function logout() {
    currentUser = null;
    localStorage.removeItem("currentUser");
    document.getElementById("mainApp").classList.add("hidden");
    document.getElementById("authModal").classList.remove("hidden");
    document.getElementById("authTitle").textContent = "Sign In";
    document.getElementById("toggleAuthBtn").textContent = "Don't have an account? Sign Up";
    document.getElementById("signInBtn").classList.remove("hidden");
    document.getElementById("signUpBtn").classList.add("hidden");
    document.getElementById("authMessage").classList.add("hidden");
    document.getElementById("usernameInput").value = "";
    document.getElementById("passwordInput").value = "";
}

// Change theme
function changeTheme() {
    const theme = document.getElementById("themeSelect").value;
    document.body.className = `min-h-screen flex flex-col items-center p-2 font-sans theme-${theme}`;
    localStorage.setItem("theme", theme);
}

// Update topic suggestions
function updateTopicSuggestions() {
    const subject = document.getElementById("subjectInput").value;
    const datalist = document.getElementById("topicSuggestions");
    datalist.innerHTML = topicSuggestions[subject].map(t => `<option value="${t}">`).join("");
}

// Render timetable
function renderTimetable() {
    const timetable = document.getElementById("timetable");
    const filterDate = document.getElementById("filterDate").value || new Date().toISOString().slice(0, 10);
    timetable.innerHTML = "";
    const filteredTasks = tasks.filter(task => task.date === filterDate && task.subject === currentSubject);
    filteredTasks.sort((a, b) => a.time.localeCompare(b.time));
    if (filteredTasks.length === 0) {
        timetable.innerHTML = `<p class="text-gray-500">No tasks for ${currentSubject} on this date.</p>`;
    }
    filteredTasks.forEach(task => {
        const taskDiv = document.createElement("div");
        taskDiv.className = `task-card p-3 border rounded-lg flex justify-between items-center ${task.completed ? "bg-green-100" : "bg-white"} ${currentSubject === "Quant" ? "border-blue-300" : currentSubject === "English" ? "border-green-300" : currentSubject === "Reasoning" ? "border-yellow-300" : "border-red-300"} hover:shadow-md transition`;
        taskDiv.innerHTML = `
            <div>
                <p class="font-medium">${task.description}</p>
                <p class="text-sm text-gray-600">${task.topic} | ${task.time} | ${task.duration} hrs${task.mockScore ? ` | Score: ${task.mockScore}` : ""}</p>
            </div>
            <div class="flex space-x-2">
                <input type="checkbox" ${task.completed ? "checked" : ""} onchange="toggleTask('${task.id}')">
                <button onclick="deleteTask('${task.id}')" class="text-red-500 hover:text-red-700">Delete</button>
            </div>
        `;
        timetable.appendChild(taskDiv);
    });
    updateRevisionPlanner();
    showDailyActivity();
}

// Generate unique ID
function generateId() {
    return 'xxxxxxxx-xxxx-4xxx-yxxx-xxxxxxxxxxxx'.replace(/[xy]/g, c => {
        const r = Math.random() * 16 | 0, v = c === 'x' ? r : (r & 0x3 | 0x8);
        return v.toString(16);
    });
}

// Add new task
function addTask() {
    const taskInput = document.getElementById("taskInput");
    const subjectInput = document.getElementById("subjectInput");
    const topicInput = document.getElementById("topicInput");
    const timeInput = document.getElementById("timeInput");
    const dateInput = document.getElementById("dateInput");
    const durationInput = document.getElementById("durationInput");
    const today = new Date().toISOString().slice(0, 10);
    if (taskInput.value && subjectInput.value && topicInput.value && timeInput.value && dateInput.value && durationInput.value) {
        if (dateInput.value < today) {
            alert("Cannot add tasks for past dates!");
            return;
        }
        tasks.push({
            id: generateId(),
            description: taskInput.value,
            subject: subjectInput.value,
            topic: topicInput.value,
            time: timeInput.value,
            date: dateInput.value,
            duration: parseFloat(durationInput.value),
            completed: false,
            mockScore: null
        });
        saveUserData();
        taskInput.value = topicInput.value = timeInput.value = durationInput.value = "";
        renderTimetable();
    } else {
        alert("Please fill all fields!");
    }
}

// Toggle task completion
function toggleTask(id) {
    const task = tasks.find(t => t.id === id);
    task.completed = !task.completed;
    if (task.completed) {
        if (task.description.toLowerCase().includes("mock test")) {
            let score = prompt("Enter your mock test score (e.g., 165/200):", "");
            if (score) {
                task.mockScore = score;
            } else {
                task.mockScore = "Not scored";
            }
        }
        studyRecords[task.subject].push({
            date: task.date,
            topic: task.topic,
            duration: task.duration,
            mockScore: task.mockScore
        });
        saveUserData();
        stopAlarm();
    }
    saveUserData();
    renderTimetable();
}

// Delete task
function deleteTask(id) {
    tasks = tasks.filter(t => t.id !== id);
    saveUserData();
    renderTimetable();
}

// Play alarm
function playAlarm() {
    const alarm = document.getElementById("alarmSound");
    alarm.play().catch(err => console.log("Audio play failed:", err));
}

// Stop alarm
function stopAlarm() {
    const alarm = document.getElementById("alarmSound");
    alarm.pause();
    alarm.currentTime = 0;
}

// Pomodoro timer
let pomodoroInterval = null;
let pomodoroSeconds = 0;
function startPomodoro() {
    if (pomodoroInterval) return;
    pomodoroSeconds = 25 * 60;
    stopAlarm();
    pomodoroInterval = setInterval(() => {
        pomodoroSeconds--;
        const minutes = Math.floor(pomodoroSeconds / 60);
        const seconds = pomodoroSeconds % 60;
        document.getElementById("pomodoroTimer").textContent = `Pomodoro: ${minutes}:${seconds < 10 ? "0" : ""}${seconds}`;
        if (pomodoroSeconds <= 0) {
            clearInterval(pomodoroInterval);
            pomodoroInterval = null;
            document.getElementById("pomodoroTimer").textContent = "Pomodoro Done! Take a 5-min break.";
            alert("Pomodoro session complete! Take a 5-min break.");
        }
    }, 1000);
}

// Check for missed tasks
function checkMissedTasks() {
    if (pomodoroInterval) return;
    const now = new Date();
    const currentDate = now.toISOString().slice(0, 10);
    const currentTime = now.toTimeString().slice(0, 5);
    tasks.forEach(task => {
        if (!task.completed && task.date === currentDate && task.time < currentTime) {
            if (Notification.permission === "granted") {
                new Notification(`Missed Task: ${task.description}`, {
                    body: `Scheduled for ${task.time} (${task.subject}). Mark it as done!`,
                    icon: "https://via.placeholder.com/32"
                });
            }
            playAlarm();
        }
    });
}

// Show subject-specific content
function showSubject(subject) {
    currentSubject = subject;
    document.querySelectorAll(".flex.space-x-2 button").forEach(btn => {
        btn.className = `px-4 py-2 font-semibold ${btn.textContent === subject ? `border-b-2 border-${subject === "Quant" ? "blue" : subject === "English" ? "green" : subject === "Reasoning" ? "yellow" : "red"}-600 text-${subject === "Quant" ? "blue" : subject === "English" ? "green" : subject === "Reasoning" ? "yellow" : "red"}-600` : "text-gray-600 hover:border-b-2 hover:border-gray-600"}`;
    });
    document.getElementById("subjectInput").value = subject;
    updateTopicSuggestions();
    renderTimetable();
    showDailyRecords();
}

// Show daily records
function showDailyRecords() {
    const recordsDiv = document.getElementById("records");
    const today = new Date().toISOString().slice(0, 10);
    const records = studyRecords[currentSubject].filter(r => r.date === today);
    recordsDiv.innerHTML = `<h3 class="font-semibold">${currentSubject} Daily Records (${today})</h3>`;
    if (records.length === 0) {
        recordsDiv.innerHTML += `<p class="text-gray-500">No ${currentSubject} records for today.</p>`;
        return;
    }
    const summary = aggregateRecords(records);
    recordsDiv.innerHTML += `
        <p><strong>Total Hours</strong>: ${summary.duration} hrs</p>
        <div class="w-full bg-gray-200 h-2 rounded-full mt-2">
            <div class="bg-${currentSubject === "Quant" ? "blue" : currentSubject === "English" ? "green" : currentSubject === "Reasoning" ? "yellow" : "red"}-500 h-2 rounded-full" style="width: ${Math.min(summary.duration / 10 * 100, 100)}%"></div>
        </div>
        <ul class="list-disc pl-5 mt-2">
            ${summary.topics.map(t => `<li>${t.topic} (${t.duration} hrs)</li>`).join("")}
        </ul>
    `;
    const mockTests = records.filter(r => r.mockScore);
    if (mockTests.length > 0) {
        recordsDiv.innerHTML += `
            <h4 class="font-semibold mt-4">Mock Test Scores</h4>
            <ul class="list-disc pl-5">
                ${mockTests.map(m => `<li>${m.date}: ${m.mockScore}</li>`).join("")}
            </ul>
        `;
    }
}

// Show weekly records
function showWeeklyRecords() {
    const recordsDiv = document.getElementById("records");
    const today = new Date();
    const weekStart = new Date(today.setDate(today.getDate() - today.getDay()));
    const weekEnd = new Date(today.setDate(today.getDate() + 6));
    const records = studyRecords[currentSubject].filter(r => {
        const recordDate = new Date(r.date);
        return recordDate >= weekStart && recordDate <= weekEnd;
    });
    recordsDiv.innerHTML = `<h3 class="font-semibold">${currentSubject} Weekly Records (${weekStart.toISOString().slice(0, 10)} to ${weekEnd.toISOString().slice(0, 10)})</h3>`;
    if (records.length === 0) {
        recordsDiv.innerHTML += `<p class="text-gray-500">No ${currentSubject} records for this week.</p>`;
        return;
    }
    const summary = aggregateRecords(records);
    recordsDiv.innerHTML += `
        <p><strong>Total Hours</strong>: ${summary.duration} hrs</p>
        <div class="w-full bg-gray-200 h-2 rounded-full mt-2">
            <div class="bg-${currentSubject === "Quant" ? "blue" : currentSubject === "English" ? "green" : currentSubject === "Reasoning" ? "yellow" : "red"}-500 h-2 rounded-full" style="width: ${Math.min(summary.duration / 70 * 100, 100)}%"></div>
        </div>
        <ul class="list-disc pl-5 mt-2">
            ${summary.topics.map(t => `<li>${t.topic} (${t.duration} hrs)</li>`).join("")}
        </ul>
    `;
    const mockTests = records.filter(r => r.mockScore);
    if (mockTests.length > 0) {
        recordsDiv.innerHTML += `
            <h4 class="font-semibold mt-4">Mock Test Scores</h4>
            <ul class="list-disc pl-5">
                ${mockTests.map(m => `<li>${m.date}: ${m.mockScore}</li>`).join("")}
            </ul>
        `;
    }
}

// Show monthly records
function showMonthlyRecords() {
    const recordsDiv = document.getElementById("records");
    const today = new Date();
    const monthStart = new Date(today.getFullYear(), today.getMonth(), 1);
    const monthEnd = new Date(today.getFullYear(), today.getMonth() + 1, 0);
    const records = studyRecords[currentSubject].filter(r => {
        const recordDate = new Date(r.date);
        return recordDate >= monthStart && recordDate <= monthEnd;
    });
    recordsDiv.innerHTML = `<h3 class="font-semibold">${currentSubject} Monthly Records (${monthStart.toISOString().slice(0, 10)} to ${monthEnd.toISOString().slice(0, 10)})</h3>`;
    if (records.length === 0) {
        recordsDiv.innerHTML += `<p class="text-gray-500">No ${currentSubject} records for this month.</p>`;
        return;
    }
    const summary = aggregateRecords(records);
    recordsDiv.innerHTML += `
        <p><strong>Total Hours</strong>: ${summary.duration} hrs</p>
        <div class="w-full bg-gray-200 h-2 rounded-full mt-2">
            <div class="bg-${currentSubject === "Quant" ? "blue" : currentSubject === "English" ? "green" : currentSubject === "Reasoning" ? "yellow" : "red"}-500 h-2 rounded-full" style="width: ${Math.min(summary.duration / 300 * 100, 100)}%"></div>
        </div>
        <ul class="list-disc pl-5 mt-2">
            ${summary.topics.map(t => `<li>${t.topic} (${t.duration} hrs)</li>`).join("")}
        </ul>
    `;
    const mockTests = records.filter(r => r.mockScore);
    if (mockTests.length > 0) {
        recordsDiv.innerHTML += `
            <h4 class="font-semibold mt-4">Mock Test Scores</h4>
            <ul class="list-disc pl-5">
                ${mockTests.map(m => `<li>${m.date}: ${m.mockScore}</li>`).join("")}
            </ul>
        `;
    }
}

// Aggregate records
function aggregateRecords(records) {
    const summary = { duration: 0, topics: [] };
    records.forEach(r => {
        const topicIndex = summary.topics.findIndex(t => t.topic === r.topic);
        if (topicIndex === -1) {
            summary.topics.push({ topic: r.topic, duration: r.duration });
        } else {
            summary.topics[topicIndex].duration += r.duration;
        }
        summary.duration += r.duration;
    });
    return summary;
}

// Calculate rating based on task completion
function calculateRating(tasksToConsider, period) {
    if (tasksToConsider.length === 0) return { rating: 0, label: "No tasks" };
    
    const totalTasks = tasksToConsider.length;
    const completedTasks = tasksToConsider.filter(t => t.completed).length;
    const completionRatio = totalTasks === 0 ? 0 : completedTasks / totalTasks;
    
    const rating = Math.round(completionRatio * 10) || 1;
    let label;
    if (rating <= 5) label = "Bad";
    else if (rating <= 8) label = "Good";
    else label = "Excellent";
    
    return { rating, label };
}

// Show daily activity
function showDailyActivity() {
    const activityDiv = document.getElementById("activity");
    const today = new Date().toISOString().slice(0, 10);
    const dailyTasks = tasks.filter(t => t.date === today);
    const totalTasks = dailyTasks.length;
    const completedTasks = dailyTasks.filter(t => t.completed).length;
    const completionPercentage = totalTasks === 0 ? 0 : Math.round((completedTasks / totalTasks) * 100);
    const ratingInfo = calculateRating(dailyTasks, "daily");
    
    activityDiv.innerHTML = `
        <h3 class="font-semibold">Daily Activity (${today})</h3>
        <p><strong>Total Tasks Added</strong>: ${totalTasks}</p>
        <p><strong>Tasks Completed</strong>: ${completedTasks}</p>
        <p><strong>Completion Rate</strong>: ${completionPercentage}%</p>
        <p><strong>Rating</strong>: ${ratingInfo.rating}/10 (${ratingInfo.label})</p>
    `;
}

// Show weekly activity
function showWeeklyActivity() {
    const activityDiv = document.getElementById("activity");
    const today = new Date();
    const weekStart = new Date(today.setDate(today.getDate() - today.getDay()));
    const weekEnd = new Date(today.setDate(today.getDate() + 6));
    const weeklyTasks = tasks.filter(t => {
        const taskDate = new Date(t.date);
        return taskDate >= weekStart && taskDate <= weekEnd;
    });
    const totalTasks = weeklyTasks.length;
    const completedTasks = weeklyTasks.filter(t => t.completed).length;
    const completionPercentage = totalTasks === 0 ? 0 : Math.round((completedTasks / totalTasks) * 100);
    const ratingInfo = calculateRating(weeklyTasks, "weekly");
    
    activityDiv.innerHTML = `
        <h3 class="font-semibold">Weekly Activity (${weekStart.toISOString().slice(0, 10)} to ${weekEnd.toISOString().slice(0, 10)})</h3>
        <p><strong>Total Tasks Added</strong>: ${totalTasks}</p>
        <p><strong>Tasks Completed</strong>: ${completedTasks}</p>
        <p><strong>Completion Rate</strong>: ${completionPercentage}%</p>
        <p><strong>Rating</strong>: ${ratingInfo.rating}/10 (${ratingInfo.label})</p>
    `;
}

// Show monthly activity
function showMonthlyActivity() {
    const activityDiv = document.getElementById("activity");
    const today = new Date();
    const monthStart = new Date(today.getFullYear(), today.getMonth(), 1);
    const monthEnd = new Date(today.getFullYear(), today.getMonth() + 1, 0);
    const monthlyTasks = tasks.filter(t => {
        const taskDate = new Date(t.date);
        return taskDate >= monthStart && taskDate <= monthEnd;
    });
    const totalTasks = monthlyTasks.length;
    const completedTasks = monthlyTasks.filter(t => t.completed).length;
    const completionPercentage = totalTasks === 0 ? 0 : Math.round((completedTasks / totalTasks) * 100);
    const ratingInfo = calculateRating(monthlyTasks, "monthly");
    
    activityDiv.innerHTML = `
        <h3 class="font-semibold">Monthly Activity (${monthStart.toISOString().slice(0, 10)} to ${monthEnd.toISOString().slice(0, 10)})</h3>
        <p><strong>Total Tasks Added</strong>: ${totalTasks}</p>
        <p><strong>Tasks Completed</strong>: ${completedTasks}</p>
        <p><strong>Completion Rate</strong>: ${completionPercentage}%</p>
        <p><strong>Rating</strong>: ${ratingInfo.rating}/10 (${ratingInfo.label})</p>
    `;
}

// Update revision planner
function updateRevisionPlanner() {
    const revisionDiv = document.getElementById("revisionPlanner");
    const today = new Date();
    const isSunday = today.getDay() === 0;
    const isFirstOfMonth = today.getDate() === 1;
    if (!isSunday && !isFirstOfMonth) {
        revisionDiv.innerHTML = `<p class="text-gray-500">Revision suggestions updated every Sunday (weekly) and 1st of month (monthly).</p>`;
        return;
    }
    const records = studyRecords[currentSubject];
    const summary = aggregateRecords(records);
    const suggestions = summary.topics.sort((a, b) => a.duration - b.duration).slice(0, 2);
    revisionDiv.innerHTML = `<h3 class="font-semibold">${currentSubject} ${isSunday ? "Weekly" : "Monthly"} Revision Suggestions</h3>`;
    if (suggestions.length === 0) {
        revisionDiv.innerHTML += `<p class="text-gray-500">No ${currentSubject} records yet. Complete tasks to get suggestions.</p>`;
        return;
    }
    suggestions.forEach(s => {
        revisionDiv.innerHTML += `<p>Revise ${s.topic} (${s.duration} hrs studied)</p>`;
    });
}

// Download records
function downloadRecords() {
    const content = JSON.stringify(studyRecords[currentSubject], null, 2);
    const blob = new Blob([content], { type: "text/plain" });
    const url = URL.createObjectURL(blob);
    const a = document.createElement("a");
    a.href = url;
    a.download = `${currentSubject}_study_records.txt`;
    a.click();
    URL.revokeObjectURL(url);
}

// Run checks and initial setup
setInterval(checkMissedTasks, 60000);
