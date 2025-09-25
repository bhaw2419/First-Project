<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
    <title>OMR Evaluation System</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- PDF.js CDN -->
    <input id="file-upload" type="file" accept="image/png, image/jpeg, image/webp">


    <style>
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;500;700&display=swap');
        body {
            font-family: 'Inter', sans-serif;
            background-color: #f3f4f6;
            display: flex;
            justify-content: center;
            align-items: flex-start;
            min-height: 100vh;
            padding: 2rem;
        }
        .container {
            width: 100%;
            max-width: 1200px;
            background-color: white;
            padding: 2rem;
            border-radius: 1rem;
            box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
        }
        .omr-bubble {
    width: 28px;
    height: 28px;
    border-radius: 50%;
    border: 2px solid #d1d5db;
    display: flex;
    align-items: center;
    justify-content: center;
    font-weight: 500;
    transition: all 0.15s ease-in-out;
    user-select: none;
    cursor: default;
    flex-shrink: 0;
}
.omr-bubble.correct {
    background-color: #22c55e;
    border-color: #22c55e;
    color: white;
}
.omr-bubble.incorrect {
    background-color: #ef4444;
    border-color: #ef4444;
    color: white;
}
.omr-bubble.selected {
    background-color: #1e40af;
    border-color: #1e40af;
    color: white;
}
.omr-bubble.correct-dashed {
    border-style: dashed;
    border-color: #22c55e;
}

/* ✅ Subject grid fits container and avoids overflow */
.subject-grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
    gap: 1.25rem;
    width: 100%;
    margin-top: 1rem;
}

/* ✅ Subject card styling */
.subject-grid > div {
    display: flex;
    flex-direction: column;
    background: #ffffff;
    border-radius: 0.75rem;
    padding: 1rem;
    border: 1px solid #d1d5db;
    box-shadow: 0 2px 6px rgba(0, 0, 0, 0.05);
    max-height: 550px;  /* prevents overflow */
    overflow-y: auto;   /* adds scroll inside card if too many questions */
}

/* ✅ Question row styling */
.subject-grid .flex.items-center {
    display: flex;
    align-items: center;
    justify-content: space-between;
    gap: 0.5rem;
    margin-bottom: 0.25rem;
}

/* ✅ Responsive layout on small screens */
@media (max-width: 768px) {
    .subject-grid {
        grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
    }
}

        #video-stream, #image-preview {
            max-width: 100%;
            max-height: 400px;
            border-radius: 0.75rem;
            margin-top: 1rem;
            border: 1px solid #e5e7eb;
            object-fit: contain;
        }
        .file-upload-section {
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            padding: 2rem;
            border: 2px dashed #d1d5db;
            border-radius: 0.75rem;
            text-align: center;
            cursor: pointer;
            transition: border-color 0.2s;
        }
        .file-upload-section:hover { border-color: #60a5fa; }
        .file-upload-section input[type="file"] { display: none; }
        .custom-alert { position: fixed; top: 50%; left: 50%; transform: translate(-50%, -50%); background-color: white; padding: 2rem; border-radius: 0.75rem; box-shadow: 0 4px 20px rgba(0,0,0,0.2); z-index: 1000; width: 90%; max-width: 400px; text-align: center; }
        .overlay { position: fixed; top:0; left:0; width:100%; height:100%; background: rgba(0,0,0,0.5); z-index: 999; }
        .report-list { max-height: 220px; overflow:auto; padding-right: 8px; }
    </style>
</head>
<body class="p-4 md:p-8">
    <!-- Keep all your previous code exactly same until evaluateScores() -->
<script>
    // ... all previous code remains unchanged above ...

    // Evaluates the scores and displays results
    function evaluateScores(studentAnswers, correctAnswerKey) {
        let totalScore = 0;
        let subjectScores = Array(NUM_SUBJECTS).fill(0);
        let attemptedCount = 0;
        let unattemptedCount = 0;
        
        // Loop through each question to evaluate
        for (let i = 1; i <= TOTAL_QUESTIONS; i++) {
            const studentAnswer = studentAnswers[i];
            const correctAnswer = correctAnswerKey[i];
            
            if (studentAnswer !== null) {
                attemptedCount++;
                if (studentAnswer === correctAnswer) {
                    totalScore += 1;
                    const subjectIndex = Math.floor((i - 1) / QUESTIONS_PER_SUBJECT);
                    subjectScores[subjectIndex] += 1;
                }
            } else {
                unattemptedCount++;
            }
        }

        // Calculate scores out of 20 and 100
        const finalSubjectScores = subjectScores.map(score => (score / QUESTIONS_PER_SUBJECT) * 20);
        const finalTotalScore = (totalScore / TOTAL_QUESTIONS) * 100;
        
        // Display results
        scoreDetailsEl.innerHTML = '';
        finalSubjectScores.forEach((score, index) => {
            const subjectName = subjectNames[index] || DEFAULT_SUBJECT_NAMES[index];
            const p = document.createElement('p');
            p.className = 'text-gray-700';
            p.innerHTML = `<span class="font-medium">${subjectName} Score:</span> ${score.toFixed(2)} / 20`;
            scoreDetailsEl.appendChild(p);
        });

        // Add attempted and unattempted counts
        const attemptedP = document.createElement('p');
        attemptedP.className = 'text-gray-700 mt-4';
        attemptedP.innerHTML = `<span class="font-medium">Questions Attempted:</span> <span class="font-bold text-blue-600">${attemptedCount}</span>`;
        scoreDetailsEl.appendChild(attemptedP);

        const unattemptedP = document.createElement('p');
        unattemptedP.className = 'text-gray-700';
        unattemptedP.innerHTML = `<span class="font-medium">Questions Not Attempted:</span> <span class="font-bold text-gray-500">${unattemptedCount}</span>`;
        scoreDetailsEl.appendChild(unattemptedP);
        
        totalScoreEl.textContent = finalTotalScore.toFixed(2);
        resultsEl.classList.remove('hidden');

        // ✅ Highlight correct/incorrect answers
        document.querySelectorAll('#omr-sheet .omr-bubble').forEach(bubble => {
            const questionNumber = parseInt(
                bubble.parentElement.parentElement.querySelector('span').textContent.split('.')[0]
            );
            const option = bubble.textContent;
            bubble.classList.remove('correct', 'incorrect');
            bubble.style.border = ''; // reset

            // Correct answer selected → green
            if (studentAnswers[questionNumber] === option && studentAnswers[questionNumber] === correctAnswerKey[questionNumber]) {
                bubble.classList.add('correct');
            } 
            // Wrong answer selected → red
            else if (studentAnswers[questionNumber] === option && studentAnswers[questionNumber] !== correctAnswerKey[questionNumber]) {
                bubble.classList.add('incorrect');
            } 
            // If student got it wrong or skipped, show correct answer with dashed border
            else if (option === correctAnswerKey[questionNumber]) {
                bubble.style.border = '2px dashed #22c55e';
            }
        });
    }

    // ... keep the rest of your code unchanged below ...
</script>

<div class="container space-y-8">
    <header class="text-center mb-8">
        <h1 class="text-4xl font-bold text-gray-800">Automated OMR Evaluator</h1>
        <p class="text-gray-500 mt-2">Deterministic evaluation with exact right/wrong marking and per-question report.</p>
    </header>

    <div class="grid grid-cols-1 lg:grid-cols-3 gap-8">
        <div class="lg:col-span-2 p-6 bg-gray-50 rounded-lg border border-gray-200">
            <h2 class="text-2xl font-semibold mb-6 text-gray-700">Student OMR Sheet</h2>
            <div id="image-input-section" class="mb-8 flex flex-col items-center">
                <video id="video-stream" class="rounded-lg shadow-md border-gray-300 border-2" autoplay playsinline></video>
                <canvas id="camera-canvas" class="hidden"></canvas>
                <label for="file-upload" class="file-upload-section mt-4 hidden">
                    <span class="text-xl font-medium text-gray-600">Click to upload an image</span>
                    <span class="text-sm text-gray-400 mt-2">.jpg, .png, or .webp accepted</span>
                    <input id="file-upload" type="file" accept="image/png, image/jpeg, image/webp">
                </label>
                <img id="image-preview" class="hidden" alt="Captured OMR Sheet">
                <p class="text-center text-sm text-gray-500 mt-4">Note: The image analysis is a simulation for now.</p>
            </div>

            <div class="flex justify-center mt-8 space-x-4">
                <button id="camera-capture-btn" class="bg-blue-600 hover:bg-blue-700 text-white font-bold py-3 px-6 rounded-full shadow-lg transform transition-transform duration-200 hover:scale-105">Capture from Camera</button>
                <button id="toggle-upload-btn" class="bg-gray-500 hover:bg-gray-600 text-white font-bold py-3 px-6 rounded-full shadow-lg transform transition-transform duration-200 hover:scale-105">Switch to Upload</button>
            </div>

            <div id="omr-sheet" class="grid subject-grid gap-6 mt-8 hidden"></div>
        </div>

        <div class="lg:col-span-1 space-y-8">
            <div class="p-6 bg-gray-50 rounded-lg border border-gray-200">
                <h2 class="text-2xl font-semibold mb-4 text-gray-700">Subject Names</h2>
                <div id="subject-name-inputs" class="space-y-4"></div>
            </div>

            <div class="p-6 bg-gray-50 rounded-lg border border-gray-200">
                <h2 class="text-2xl font-semibold mb-6 text-gray-700">Answer Key</h2>
                <div id="answer-key-input" class="grid grid-cols-2 gap-4"></div>
                <div class="mt-4 flex justify-center">
                    <button id="restore-key-btn" class="bg-gray-500 hover:bg-gray-600 text-white font-bold py-2 px-4 rounded-full shadow-md text-sm transition-transform duration-200 hover:scale-105 hidden">Restore Answer Key</button>
                </div>
            </div>

            <div id="results" class="p-6 bg-green-50 rounded-lg border border-green-200 hidden">
                <h2 class="text-2xl font-semibold mb-4 text-green-700">Evaluation Results</h2>
                <div id="score-details" class="space-y-3"></div>
                <div class="border-t border-green-200 mt-4 pt-4">
                    <p class="text-lg font-bold text-green-700">Total Score: <span id="total-score">0</span> / 100</p>
                </div>
            </div>

            <div class="p-6 bg-white rounded-lg border border-gray-200 hidden" id="per-question-report-container">
                <h3 class="text-lg font-semibold mb-2">Per-question Report</h3>
                <div id="per-question-report" class="report-list text-sm text-gray-700"></div>
            </div>

            <button id="evaluate-btn" class="w-full bg-blue-600 hover:bg-blue-700 text-white font-bold py-3 px-6 rounded-full shadow-lg transform transition-transform duration-200 hover:scale-105 hidden">Evaluate Score</button>
        </div>
    </div>
</div>

<div id="custom-alert-modal" class="hidden">
    <div class="overlay"></div>
    <div class="custom-alert">
        <h3 class="text-xl font-bold mb-4 text-gray-800">Heads Up!</h3>
        <p class="text-gray-600 mb-6" id="alert-message">This feature requires camera access. Please allow it when prompted.</p>
        <button id="alert-ok-btn" class="bg-blue-600 hover:bg-blue-700 text-white font-bold py-2 px-6 rounded-full">OK</button>
    </div>
</div>

<script>
document.addEventListener('DOMContentLoaded', () => {
    // Config
    const NUM_SUBJECTS = 5;
    const QUESTIONS_PER_SUBJECT = 20;
    const TOTAL_QUESTIONS = NUM_SUBJECTS * QUESTIONS_PER_SUBJECT;
    const OPTIONS = ['A','B','C','D'];
    const DEFAULT_SUBJECT_NAMES = ['Subject 1','Subject 2','Subject 3','Subject 4','Subject 5'];
    const LOCAL_STORAGE_KEY = 'omr_answer_key';
    const SUBJECT_NAMES_KEY = 'omr_subject_names';

    // UI refs
    const videoStreamEl = document.getElementById('video-stream');
    const imagePreviewEl = document.getElementById('image-preview');
    const cameraCanvasEl = document.getElementById('camera-canvas');
    const omrSheetEl = document.getElementById('omr-sheet');
    const answerKeyInputEl = document.getElementById('answer-key-input');
    const cameraCaptureBtn = document.getElementById('camera-capture-btn');
    const toggleUploadBtn = document.getElementById('toggle-upload-btn');
    const fileUploadEl = document.getElementById('file-upload');
    const fileUploadSectionEl = document.querySelector('.file-upload-section');
    const evaluateBtn = document.getElementById('evaluate-btn');
    const resultsEl = document.getElementById('results');
    const scoreDetailsEl = document.getElementById('score-details');
    const totalScoreEl = document.getElementById('total-score');
    const customAlertModal = document.getElementById('custom-alert-modal');
    const alertOkBtn = document.getElementById('alert-ok-btn');
    const alertMessageEl = document.getElementById('alert-message');
    const restoreKeyBtn = document.getElementById('restore-key-btn');
    const subjectNameInputsEl = document.getElementById('subject-name-inputs');
    const perQuestionContainer = document.getElementById('per-question-report-container');
    const perQuestionReportEl = document.getElementById('per-question-report');

    // State
    let answerKey = {}; // {1: 'A', 2:'C', ...}
    let subjectNames = [...DEFAULT_SUBJECT_NAMES];

    // Helpers: alerts
    function showAlert(message) {
        alertMessageEl.textContent = message;
        customAlertModal.classList.remove('hidden');
    }
    function hideAlert() { customAlertModal.classList.add('hidden'); }

    // Local storage: save / restore key
    function saveAnswerKey() {
        try {
            localStorage.setItem(LOCAL_STORAGE_KEY, JSON.stringify(answerKey));
            restoreKeyBtn.classList.remove('hidden');
        } catch(e) { console.error('Save key failed', e); }
    }
    function restoreAnswerKey() {
        try {
            const stored = localStorage.getItem(LOCAL_STORAGE_KEY);
            if (stored) {
                answerKey = JSON.parse(stored) || {};
                // update UI selects
                for (let i=1;i<=TOTAL_QUESTIONS;i++){
                    const sel = document.querySelector(`select[name="key-q${i}"]`);
                    if (sel) sel.value = answerKey[i] || '';
                }
            }
        } catch(e) { console.error('Restore key failed', e); localStorage.removeItem(LOCAL_STORAGE_KEY); }
    }

    function saveSubjectNames() {
        try { localStorage.setItem(SUBJECT_NAMES_KEY, JSON.stringify(subjectNames)); }
        catch(e) { console.error('save subject names failed', e); }
    }
    function restoreSubjectNames() {
        try {
            const stored = localStorage.getItem(SUBJECT_NAMES_KEY);
            if (stored) subjectNames = JSON.parse(stored) || subjectNames;
        } catch(e) { console.error('restore subject names failed', e); localStorage.removeItem(SUBJECT_NAMES_KEY); }
    }

    // Generate subject name inputs
    function generateSubjectNameInputs(){
        subjectNameInputsEl.innerHTML='';
        restoreSubjectNames();
        for (let i=0;i<NUM_SUBJECTS;i++){
            const div = document.createElement('div');
            div.className = 'flex items-center space-x-2';
            div.innerHTML = `<span class="font-bold w-6 text-gray-600">${i+1}.</span>`;
            const input = document.createElement('input');
            input.type='text';
            input.placeholder = DEFAULT_SUBJECT_NAMES[i];
            input.className = 'flex-1 p-2 border border-gray-300 rounded-lg focus:outline-none focus:border-blue-500';
            input.value = subjectNames[i] || '';
            input.oninput = (e)=> { subjectNames[i]=e.target.value; saveSubjectNames(); };
            div.appendChild(input);
            subjectNameInputsEl.appendChild(div);
        }
    }

    // Generate answer key UI
    function generateAnswerKeyInput(){
        answerKeyInputEl.innerHTML='';
        restoreAnswerKey();
        for (let i=1;i<=TOTAL_QUESTIONS;i++){
            const qDiv = document.createElement('div');
            qDiv.className = 'flex items-center space-x-2';
            qDiv.innerHTML = `<span class="font-bold w-6 text-gray-600">${i}.</span>`;
            const selectEl = document.createElement('select');
            selectEl.className = 'flex-1 p-2 border border-gray-300 rounded-lg focus:outline-none focus:border-blue-500';
            selectEl.name = `key-q${i}`;
            selectEl.innerHTML = '<option value="">-</option>';
            OPTIONS.forEach(opt => { selectEl.innerHTML += `<option value="${opt}">${opt}</option>`; });
            if (answerKey[i]) selectEl.value = answerKey[i];
            selectEl.onchange = (e) => {
                const val = e.target.value;
                if (val) answerKey[i] = val;
                else delete answerKey[i];
                saveAnswerKey();
            };
            qDiv.appendChild(selectEl);
            answerKeyInputEl.appendChild(qDiv);
        }
        if (localStorage.getItem(LOCAL_STORAGE_KEY)) restoreKeyBtn.classList.remove('hidden');
        else restoreKeyBtn.classList.add('hidden');
    }

    // Start camera (optional)
    async function startCamera(){
        try {
            const stream = await navigator.mediaDevices.getUserMedia({ video: { facingMode: 'environment' } });
            videoStreamEl.srcObject = stream;
            videoStreamEl.classList.remove('hidden');
            imagePreviewEl.classList.add('hidden');
            if (fileUploadSectionEl) fileUploadSectionEl.classList.add('hidden');
            cameraCaptureBtn.classList.remove('hidden');
            toggleUploadBtn.textContent = 'Switch to Upload';
        } catch (err) {
            console.warn('Camera start failed: ', err);
            // show upload fallback UI
            videoStreamEl.classList.add('hidden');
            cameraCaptureBtn.classList.add('hidden');
            if (fileUploadSectionEl) fileUploadSectionEl.classList.remove('hidden');
        }
    }

    // Simulated analysis (same as before but deterministic if you want)
    // For repeatable testing, you might replace Math.random with deterministic behavior.
   // Store simulated answers so they don't change on every evaluation


// Simulates the OMR analysis process (deterministic after first run)
// Store simulated answers so they don't change on every evaluation
let savedStudentAnswers = null;

// Simulates the OMR analysis process (deterministic after first run)
function simulateAnalysis(key) {
    // If we already simulated once, reuse the same answers
    if (savedStudentAnswers) return savedStudentAnswers;

    const studentAnswers = {};
    for (let i = 1; i <= TOTAL_QUESTIONS; i++) {
        if (key[i] && Math.random() < 0.05) {
            studentAnswers[i] = null; // Unattempted
        } else if (key[i] && Math.random() < 0.9) {
            studentAnswers[i] = key[i]; // Correct
        } else if (key[i]) {
            const incorrectOptions = OPTIONS.filter(o => o !== key[i]);
            studentAnswers[i] = incorrectOptions[Math.floor(Math.random() * incorrectOptions.length)];
        } else {
            studentAnswers[i] = null;
        }
    }

    savedStudentAnswers = studentAnswers; // Save once
    return studentAnswers;
}


    // Generate OMR sheet DOM using data-q attributes
    function generateOmrSheet(studentAnswers) {
        omrSheetEl.innerHTML = '';
        for (let s=0;s<NUM_SUBJECTS;s++){
            const subjectName = subjectNames[s] || DEFAULT_SUBJECT_NAMES[s];
            const subjectDiv = document.createElement('div');
            subjectDiv.className = 'p-4 border border-gray-300 rounded-lg bg-white shadow-sm';
            subjectDiv.innerHTML = `<h3 class="text-lg font-semibold mb-4 text-center">${subjectName}</h3>`;
            for (let q=1;q<=QUESTIONS_PER_SUBJECT;q++){
                const questionNumber = (s * QUESTIONS_PER_SUBJECT) + q;
                const questionDiv = document.createElement('div');
                questionDiv.className = 'flex items-center space-x-2 my-2';
                questionDiv.innerHTML = `<span class="font-bold w-6 text-gray-600">${questionNumber}.</span>`;
                const bubblesContainer = document.createElement('div');
                bubblesContainer.className = 'flex space-x-2';
                OPTIONS.forEach(option=>{
                    const bubble = document.createElement('div');
                    bubble.className = 'omr-bubble';
                    bubble.textContent = option;
                    // store question and option for reliable lookup later
                    bubble.dataset.q = questionNumber;
                    bubble.dataset.opt = option;
                    // Mark student's selection visually as "selected" initially (neutral)
                    if (studentAnswers[questionNumber] === option) bubble.classList.add('selected');
                    bubblesContainer.appendChild(bubble);
                });
                questionDiv.appendChild(bubblesContainer);
                subjectDiv.appendChild(questionDiv);
            }
            omrSheetEl.appendChild(subjectDiv);
        }
    }

    // Evaluate and update UI with exact right/wrong + per-question report
    function evaluateScores(studentAnswers, correctAnswerKey) {
        // Validate key completeness
        const providedCount = Object.keys(correctAnswerKey).filter(k => correctAnswerKey[k]).length;
        if (providedCount < TOTAL_QUESTIONS) {
            showAlert(`Answer key is incomplete. Please provide answers for all ${TOTAL_QUESTIONS} questions. Currently provided: ${providedCount}.`);
            return;
        }

        let totalCorrect = 0;
        const subjectScores = Array(NUM_SUBJECTS).fill(0);
        let attemptedCount = 0, unattemptedCount = 0;

        // Build per-question report array
        const report = [];

        for (let i=1;i<=TOTAL_QUESTIONS;i++){
            const stud = studentAnswers[i]; // null or 'A'..'D'
            const corr = correctAnswerKey[i] || null;
            let status = 'Not Attempted';
            if (stud === null || stud === undefined) {
                unattemptedCount++;
                status = 'Not Attempted';
            } else {
                attemptedCount++;
                if (stud === corr) {
                    totalCorrect++;
                    status = 'Correct';
                    const subjIdx = Math.floor((i-1)/QUESTIONS_PER_SUBJECT);
                    subjectScores[subjIdx] += 1;
                } else {
                    status = 'Incorrect';
                }
            }
            report.push({q:i, student: stud, correct: corr, status});
        }

        // Compute final subject scores (out of 20) and overall percent
        const finalSubjectScores = subjectScores.map(sc => (sc / QUESTIONS_PER_SUBJECT) * 20);
        const finalTotalScore = (totalCorrect / TOTAL_QUESTIONS) * 100;

        // Display summary
        scoreDetailsEl.innerHTML = '';
        finalSubjectScores.forEach((score, idx) => {
            const subName = subjectNames[idx] || DEFAULT_SUBJECT_NAMES[idx];
            const p = document.createElement('p');
            p.className = 'text-gray-700';
            p.innerHTML = `<span class="font-medium">${subName} Score:</span> ${score.toFixed(2)} / 20`;
            scoreDetailsEl.appendChild(p);
        });
        const attemptedP = document.createElement('p');
        attemptedP.className = 'text-gray-700 mt-4';
        attemptedP.innerHTML = `<span class="font-medium">Questions Attempted:</span> <span class="font-bold text-blue-600">${attemptedCount}</span>`;
        scoreDetailsEl.appendChild(attemptedP);
        const unattemptedP = document.createElement('p');
        unattemptedP.className = 'text-gray-700';
        unattemptedP.innerHTML = `<span class="font-medium">Questions Not Attempted:</span> <span class="font-bold text-gray-500">${unattemptedCount}</span>`;
        scoreDetailsEl.appendChild(unattemptedP);
        totalScoreEl.textContent = finalTotalScore.toFixed(2);
        resultsEl.classList.remove('hidden');

        // Update OMR bubble highlighting deterministically using data attributes
        document.querySelectorAll('#omr-sheet .omr-bubble').forEach(b => {
            // reset classes and inline styles
            b.classList.remove('correct','incorrect','selected','correct-dashed');
            b.style.borderStyle = ''; // remove dashed if any
            const qnum = parseInt(b.dataset.q, 10);
            const opt = b.dataset.opt;
            const stud = studentAnswers[qnum];
            const corr = correctAnswerKey[qnum];

            if (stud !== null && stud === opt && stud === corr) {
                // Student selected this option and it's correct
                b.classList.add('correct');
            } else if (stud !== null && stud === opt && stud !== corr) {
                // Student selected this option but it's incorrect
                b.classList.add('incorrect');
            } else if (opt === corr && (stud === null || stud !== corr)) {
                // This is the correct answer but student didn't select it (either unattempted or selected wrong)
                // show dashed border
                b.classList.add('correct-dashed');
            } else {
                // neutral / unselected
                // leave it as default
            }
        });

        // Render the per-question report for exact right/wrong
        perQuestionReportEl.innerHTML = '';
        report.forEach(r => {
            const div = document.createElement('div');
            div.className = 'flex justify-between items-center py-1 border-b border-gray-100';
            const left = document.createElement('div');
            left.innerHTML = `<span class="font-semibold">Q${r.q}.</span> Student: <span class="font-medium">${r.student || '-'}</span>`;
            const right = document.createElement('div');
            right.innerHTML = `Correct: <span class="font-medium">${r.correct || '-'}</span> — <span class="${r.status==='Correct' ? 'text-green-600 font-semibold' : (r.status==='Incorrect' ? 'text-red-600 font-semibold' : 'text-gray-500') }">${r.status}</span>`;
            div.appendChild(left);
            div.appendChild(right);
            perQuestionReportEl.appendChild(div);
        });
        perQuestionContainer.classList.remove('hidden');
        omrSheetEl.classList.remove('hidden');
    }

    // Event listeners
    cameraCaptureBtn.addEventListener('click', () => {
        const ctx = cameraCanvasEl.getContext('2d');
        cameraCanvasEl.width = videoStreamEl.videoWidth;
        cameraCanvasEl.height = videoStreamEl.videoHeight;
        ctx.drawImage(videoStreamEl, 0, 0, cameraCanvasEl.width, cameraCanvasEl.height);
        const dataUrl = cameraCanvasEl.toDataURL('image/png');
        imagePreviewEl.src = dataUrl;
        imagePreviewEl.classList.remove('hidden');
        videoStreamEl.classList.add('hidden');
        if (videoStreamEl.srcObject) videoStreamEl.srcObject.getTracks().forEach(t => t.stop());
        evaluateBtn.classList.remove('hidden');
    });

    toggleUploadBtn.addEventListener('click', () => {
        if (videoStreamEl.srcObject) videoStreamEl.srcObject.getTracks().forEach(t=>t.stop());
        videoStreamEl.classList.add('hidden');
        cameraCaptureBtn.classList.add('hidden');
        if (fileUploadSectionEl) fileUploadSectionEl.classList.remove('hidden');
        toggleUploadBtn.textContent = 'Switch to Camera';
    });

    if (fileUploadEl) {
        fileUploadEl.addEventListener('change', async (e) => {
    const file = e.target.files[0];
    if (!file) return;

    const fileType = file.type;

    if (fileType === "application/pdf") {
        // Handle PDF file with PDF.js
        const pdfData = await file.arrayBuffer();
        const pdf = await pdfjsLib.getDocument({ data: pdfData }).promise;

        // Clear any previous preview
        imagePreviewEl.classList.add('hidden');
        omrSheetEl.classList.add('hidden');

        const pageContainer = document.createElement('div');
        pageContainer.className = "space-y-4 w-full mt-4";

        for (let i = 1; i <= pdf.numPages; i++) {
            const page = await pdf.getPage(i);
            const viewport = page.getViewport({ scale: 1.5 });

            const canvas = document.createElement("canvas");
            const context = canvas.getContext("2d");
            canvas.width = viewport.width;
            canvas.height = viewport.height;

            await page.render({ canvasContext: context, viewport }).promise;

            // Create a clickable image preview
            const img = document.createElement("img");
            img.src = canvas.toDataURL();
            img.className = "rounded-lg border border-gray-300 cursor-pointer hover:shadow-lg hover:scale-105 transition-all";
            img.style.maxWidth = "100%";

            img.addEventListener("click", () => {
                imagePreviewEl.src = img.src;
                imagePreviewEl.classList.remove('hidden');
                evaluateBtn.classList.remove('hidden');
                pageContainer.remove(); // hide previews after selection
            });

            pageContainer.appendChild(img);
        }

        fileUploadSectionEl.appendChild(pageContainer);

    } else {
        // Handle normal image as before
        const reader = new FileReader();
        reader.onload = (event) => {
            imagePreviewEl.src = event.target.result;
            imagePreviewEl.classList.remove('hidden');
            evaluateBtn.classList.remove('hidden');
        };
        reader.readAsDataURL(file);
    }
});

    }

    evaluateBtn.addEventListener('click', () => {
        if (!imagePreviewEl.src) { showAlert('Please capture or upload an OMR sheet image first.'); return; }
        // validate complete key provided
        const providedCount = Object.keys(answerKey).filter(k => answerKey[k]).length;
        if (providedCount < TOTAL_QUESTIONS) { showAlert(`Please enter the full answer key (${TOTAL_QUESTIONS} answers). Currently provided: ${providedCount}.`); return; }

        // simulate / or call real analysis
        const simulatedAnswers = simulateAnalysis(answerKey);
        generateOmrSheet(simulatedAnswers);
        evaluateScores(simulatedAnswers, answerKey);
        omrSheetEl.classList.remove('hidden');
        resultsEl.classList.remove('hidden');
    });

    alertOkBtn.addEventListener('click', hideAlert);

    restoreKeyBtn.addEventListener('click', () => {
        restoreAnswerKey();
        showAlert('Answer key restored from local storage.');
    });

    // initialization
    generateSubjectNameInputs();
    generateAnswerKeyInput();
    // Try camera but it's optional
    startCamera();

    // Make the evaluate button visible only when there's preview or camera capture
    // (you'll toggle it by capture/upload actions)
});
</script>
</body>
</html>

