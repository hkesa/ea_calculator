<!DOCTYPE html>
<html lang="zh-HK">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>外傭薪酬及假期計算機</title>
    <!-- Tailwind CSS for styling and RWD -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- FontAwesome for Icons -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <!-- jsPDF and html2canvas for PDF export -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/html2canvas/1.4.1/html2canvas.min.js"></script>

    <style>
        /* Custom Colors */
        .bg-royal-blue { background-color: #4169E1; }
        .text-royal-blue { color: #4169E1; }
        .bg-light-yellow { background-color: #FFFFE0; }
        
        /* Font Size Adjustment Request (+5px from base approx 16px -> 21px) */
        .text-enlarged {
            font-size: 21px; 
            line-height: 1.6;
        }

        /* Specifically requested +5px larger elements (Base 21px + 5px = 26px or relative to context) */
        .text-plus-5 {
            font-size: 26px !important;
            line-height: 1.5;
        }
        
        /* Specific font size for clear buttons as requested (+5px relative to base button size) */
        .btn-clear-large {
            font-size: 19px !important; /* Base usually 14px -> +5px */
        }

        /* Utility */
        .input-box {
            border: 1px solid black;
            padding: 0.5rem;
            width: 100%;
            border-radius: 0.25rem;
            margin-bottom: 0.5rem; /* Spacing for vertical layout */
        }
        .input-box:focus {
            outline: 2px solid #4169E1;
            border-color: #4169E1;
        }
        
        /* Updated Result Box Style with Black Border */
        .calc-result-box {
            background-color: #FFFFE0;
            border: 1px solid black; /* Changed to black as requested */
            padding: 0.5rem;
            width: 100%;
            border-radius: 0.25rem;
            min-height: 46px; 
            display: flex;
            align-items: center;
            margin-bottom: 0.5rem;
        }

        .calc-result-box-plain {
             background-color: #FFFFE0;
             border: 1px solid #e5e7eb;
             padding: 0.5rem;
             width: 100%;
             border-radius: 0.25rem;
             min-height: 46px; 
             display: flex;
             align-items: center;
             margin-bottom: 0.5rem;
        }

        .validation-error {
            background-color: #fee2e2; /* red-100 */
            color: #991b1b; /* red-800 */
            padding: 0.5rem;
            font-size: 22px; /* Requested +5px */
            margin-top: 0.25rem;
            border-radius: 0.25rem;
            display: none; 
        }
        
        /* Print/PDF styles */
        .preview-page {
            font-family: 'Times New Roman', serif;
            background: white;
            padding: 40px;
            max-width: 800px;
            margin: 0 auto;
            border: 1px solid #ccc;
            font-size: 20px; /* Base 18 + 2px requested */
            color: black !important;
        }
        
        .preview-page * {
            color: black !important;
        }

        /* Preview Alignment Adjustments */
        .preview-amount {
            position: relative;
            top: -6px; /* Moved up 3px more from previous -3px */
        }
        
        .preview-currency {
            position: relative;
            top: -9px; /* Moved up 3px more from previous -6px */
            margin-right: 2px;
        }

        @media print {
            body * { visibility: hidden; }
            #previewSection, #previewSection * { visibility: visible; }
            #previewSection { position: absolute; left: 0; top: 0; width: 100%; border: none; }
            .no-print { display: none !important; }
        }

        /* Navigation Buttons */
        .nav-btn {
            background-color: transparent;
            color: white;
            border: 1px solid transparent;
        }
        .nav-btn.active {
            background-color: white;
            color: #4169E1; /* Royal Blue Text */
            font-weight: bold;
            border-radius: 0.25rem;
        }
    </style>
</head>
<body class="bg-gray-50 text-gray-800 font-sans">

    <!-- Navigation Bar -->
    <nav class="bg-royal-blue shadow-md sticky top-0 z-50">
        <div class="container mx-auto px-4">
            <!-- Centered Buttons -->
            <div class="flex justify-center items-center py-3 gap-4">
                <button onclick="switchTab('termination')" id="btn-termination" class="nav-btn active px-6 py-2 transition">📝 終止計算機</button>
                <button onclick="switchTab('weeks')" id="btn-weeks" class="nav-btn px-6 py-2 transition">🗓️ 4/6/8週計算機</button>
                <button onclick="switchTab('leave')" id="btn-leave" class="nav-btn px-6 py-2 transition">✈️ 放假計算機</button>
            </div>
        </div>
    </nav>

    <div class="container mx-auto px-4 py-8 mb-20">

        <!-- ========================================== -->
        <!-- TAB 1: TERMINATION CALCULATOR -->
        <!-- ========================================== -->
        <div id="tab-termination" class="tab-content block text-enlarged">
            
            <!-- PART 1: INPUT FORM -->
            <div id="inputFormSection" class="bg-white p-6 rounded-lg shadow-lg max-w-4xl mx-auto">
                <div class="flex justify-between items-center mb-6 border-b pb-2">
                    <h2 class="text-2xl font-bold flex items-center gap-3">
                        ✍🏻 填寫計算資料
                    </h2>
                    <button type="button" onclick="clearTerminationForm()" class="bg-gray-200 text-gray-700 px-4 py-1 rounded hover:bg-gray-300 btn-clear-large">
                        <i class="fa-solid fa-eraser mr-1"></i> 清除內容
                    </button>
                </div>

                <form id="termForm" onsubmit="event.preventDefault();">
                    
                    <!-- Salary -->
                    <div class="mb-6">
                        <label class="block font-semibold mb-1">合約月薪 (不用填寫$)</label>
                        <input type="number" id="salary" class="input-box" oninput="validateSalary(); calcDailyRate();" placeholder="4870">
                        <div id="err-salary" class="validation-error">你漏寫資料，請檢查和填寫所有資料</div>
                        <div id="warn-salary-low" class="validation-error text-plus-5">合約月薪應要多過4870，請檢查和填寫正確月薪</div>
                    </div>

                    <div class="mb-6">
                        <label class="block font-semibold mb-1">日薪</label>
                        <div id="dailyRate" class="calc-result-box-plain">0.000</div>
                        <p class="text-sm text-gray-500 mt-1">計算方式：合約月薪 × 12 ÷ 365</p>
                    </div>

                    <!-- Contracts -->
                    <div class="mb-6">
                        <label class="block font-semibold mb-1">這是第幾份合約 (即問前後總共有幾多份合約) 數量會影響下面的「享有年假日數」</label>
                        <select id="contractCount" class="input-box" onchange="calcLeave(); validateRequired(this);">
                            <option value="">請選擇</option>
                            <option value="1">1份</option>
                            <option value="2">2份</option>
                            <option value="3">3份</option>
                            <option value="4">4份</option>
                            <!-- Generate 5-15 -->
                            <script>
                                for(let i=5; i<=15; i++) document.write(`<option value="${i}">${i}份</option>`);
                            </script>
                        </select>
                        <div class="validation-error">你漏寫資料，請檢查和填寫所有資料</div>
                    </div>

                    <!-- Dates -->
                    <div class="mb-6">
                        <label class="block font-semibold mb-1">最後一份合約的第一天工作日</label>
                        <input type="date" id="lastContractStart" class="input-box" onblur="validateRequired(this); calcLeave(); calcWorkYears(); checkThreeMonths();">
                        <div class="validation-error">你漏寫資料，請檢查和填寫所有資料</div>
                    </div>

                    <div class="mb-6">
                        <label class="block font-semibold mb-1">上個支薪期的工資結算日</label>
                        <input type="date" id="lastPayEnd" class="input-box" onblur="validateRequired(this); calcPeriodStart(); checkLatePayment();">
                        <div class="validation-error">你漏寫資料，請檢查和填寫所有資料</div>
                    </div>

                    <div class="mb-6">
                        <label class="block font-semibold mb-1">離職月份的支薪期的第一天</label>
                        <div id="periodStart" class="calc-result-box font-bold">-</div>
                    </div>

                    <div class="mb-6">
                        <label class="block font-semibold mb-1">最後工作日 (離職日)</label>
                        <input type="date" id="lastWorkDay" class="input-box" onblur="validateRequired(this); calcWorkDays(); calcLeave(); checkThreeMonths(); checkLatePayment(); calcWorkYears();">
                        <div class="validation-error">你漏寫資料，請檢查和填寫所有資料</div>
                    </div>

                    <!-- Work Days Result -->
                    <div class="mb-6">
                        <!-- Moved Late Payment Warning here -->
                        <div id="warn-latepay" class="validation-error text-plus-5 mb-2">提醒：出糧不能遲多過7天，完整月份的日數加7天。例如：2月28號+8日，就已經出糧遲多過7天。所以應先出糧其中的月薪，餘下的日數才填寫在「上個支薪期的工資結算日」</div>

                        <label class="block font-semibold mb-1">離職月份的工作日數</label>
                        <div id="workDaysResult" class="calc-result-box font-bold">-</div>
                        
                        <div id="warn-3months" class="validation-error text-plus-5">提醒：受僱未滿3個月，則沒有有薪年假，因此下面的「享有年假日數」應填寫為0日。如果想更改為0日，則在「已放年假日數」減相同的日數以互相抵消</div>
                    </div>

                    <!-- Leave Calculations -->
                    <div class="mb-6">
                        <label class="block font-semibold mb-1 text-black">享有年假日數 (按比例)</label>
                        <div id="proRataLeave" class="calc-result-box text-black font-bold">0.000</div>
                    </div>

                    <div class="mb-6">
                        <label class="block font-semibold mb-1 text-black">享有年假+假期日數 (最終)</label>
                        <div id="finalLeaveDays" class="calc-result-box text-black font-bold">0.000</div>
                    </div>

                    <div class="mb-6">
                        <label class="block font-semibold mb-1 text-black">已放年假日數</label>
                        <input type="number" id="leaveTaken" class="input-box text-black" placeholder="請輸入數字" oninput="calcFinalLeave()" onblur="validateRequired(this);">
                        <div class="validation-error">你漏寫資料，請檢查和填寫所有資料</div>
                    </div>

                    <div class="mb-6">
                        <label class="block font-semibold mb-1 text-black">未放假期日數 (即是 休息日 和 勞工假期)</label>
                        <input type="number" id="holidaysUntaken" class="input-box text-black" placeholder="請輸入數字" oninput="calcFinalLeave()" onblur="validateRequired(this);">
                        <div class="validation-error">你漏寫資料，請檢查和填寫所有資料</div>
                    </div>

                    <!-- Allowances -->
                    <div class="mb-6">
                        <label class="block font-semibold mb-1">交通津貼 (最少：100) 不用填寫$</label>
                        <input type="number" id="travelAllowance" class="input-box" value="100" onblur="validateRequired(this);">
                        <div class="validation-error">你漏寫資料，請檢查和填寫所有資料</div>
                    </div>

                    <div class="mb-6">
                        <label class="block font-semibold mb-1">回國機票</label>
                        <select id="airTicketType" class="input-box" onchange="toggleAirTicketInput()">
                            <option value="會買機票">會買機票</option>
                            <option value="已買機票">已買機票</option>
                            <option value="折現機票">折現機票</option>
                        </select>
                        <!-- Popup input for cash -->
                        <div id="airTicketCashInputDiv" class="hidden mt-2">
                            <label class="block text-sm mb-1 text-plus-5">輸入機票折現金額 (不要填寫$)</label>
                            <input type="number" id="airTicketCashValue" class="input-box" placeholder="輸入金額" onblur="validateRequired(this)">
                            <div class="validation-error">你漏寫資料，請檢查和填寫所有資料</div>
                        </div>
                    </div>

                    <!-- Notice Pay Section -->
                    <div class="mb-6 p-4 border rounded bg-gray-50">
                        <label class="block font-semibold mb-2">一個月通知期</label>
                        <select id="noticePayType" class="input-box">
                            <option value="給一個月通知期">給一個月通知期</option>
                            <option value="給一個月代通知金">給一個月代通知金</option>
                            <option value="給部分代通知金">給部分代通知金</option>
                        </select>
                        
                        <!-- Always visible input group as per requirement -->
                        <div id="noticeInputGroup" class="pl-0 mt-2">
                            <label class="block text-sm mb-1 text-plus-5">終止合約通知日期 (寫通知日期，不是寫離職日期)</label>
                            <input type="date" id="noticeDate" class="input-box" onblur="validateRequired(this); checkNoticeDate();">
                            <div class="validation-error">你漏寫資料，請檢查和填寫所有資料</div>
                            <div id="warn-notice-date" class="validation-error">通知日期是否有錯誤，請檢查</div>
                        </div>
                    </div>

                    <div class="mb-6 p-4 border rounded bg-gray-50">
                        <label class="flex items-center space-x-2 cursor-pointer mb-2">
                            <input type="checkbox" id="checkLSP" class="w-5 h-5 text-blue-600" onchange="toggleInput('lspInputGroup')">
                            <span class="font-semibold">長期服務金 (如有)</span>
                        </label>
                        <div id="lspInputGroup" class="hidden pl-0 mt-2">
                            <label class="block text-sm mb-1 text-plus-5">第一份合約的第一天工作日 (所有合約)</label>
                            <input type="date" id="firstContractDate" class="input-box" onblur="if(document.getElementById('checkLSP').checked) { validateRequired(this); calcWorkYears(); }">
                            <div class="validation-error">你漏寫資料，請檢查和填寫所有資料</div>
                            
                            <div id="workYearsDisplay" class="mt-2 p-2 bg-gray-100 rounded text-plus-5 hidden"></div>
                            
                            <div id="warn-lsp-salary" class="validation-error text-plus-5">(少於5年，不獲長期服務金)</div>
                        </div>
                    </div>

                    <!-- Others -->
                    <div class="mb-6">
                        <label class="block font-semibold mb-1">其他款項 (如有) 銀碼如果是負數就填寫「-100」(不用填寫$)</label>
                        <input type="number" id="otherPayment" class="input-box" placeholder="例如：800" value="0">
                    </div>

                    <div class="mb-6">
                        <label class="block font-semibold mb-1">備註 (如有)</label>
                        <input type="text" id="remark" class="input-box" placeholder="非必要填寫，如要填寫備註，請以英文填寫">
                    </div>

                    <!-- Details - Vertical Layout & On Top -->
                    <div class="mb-6 p-4 border rounded bg-gray-50">
                        <div class="flex items-center gap-3 mb-2">
                            <label class="flex items-center space-x-2 cursor-pointer">
                                <input type="checkbox" id="checkDetails" class="w-5 h-5 text-blue-600" onchange="toggleInput('detailsInputGroup')">
                                <span class="font-semibold">填寫合約資料 (如有)</span>
                            </label>
                        </div>
                        
                        <div id="detailsInputGroup" class="hidden mt-4 space-y-4">
                            <div>
                                <label class="block mb-1 font-semibold">僱主姓名</label>
                                <input type="text" id="employerName" class="input-box" placeholder="" onblur="validateContractDetails(this)">
                                <div class="validation-error">你漏寫資料，請檢查和填寫所有資料</div>
                            </div>
                            <div>
                                <label class="block mb-1 font-semibold">外傭姓名</label>
                                <input type="text" id="helperName" class="input-box" placeholder="" onblur="validateContractDetails(this)">
                                <div class="validation-error">你漏寫資料，請檢查和填寫所有資料</div>
                            </div>
                            <div>
                                <label class="block mb-1 font-semibold">外傭香港身份證號碼／護照號碼</label>
                                <input type="text" id="helperId" class="input-box" placeholder="" onblur="validateContractDetails(this)">
                                <div class="validation-error">你漏寫資料，請檢查和填寫所有資料</div>
                            </div>
                            <div>
                                <label class="block mb-1 font-semibold">合約號碼</label>
                                <input type="text" id="contractNo" class="input-box" placeholder="" onblur="validateContractDetails(this)">
                                <div class="validation-error">你漏寫資料，請檢查和填寫所有資料</div>
                            </div>
                            <div>
                                <label class="block mb-1 font-semibold">款項收妥日期</label>
                                <input type="date" id="paymentDate" class="input-box" onblur="validateContractDetails(this)">
                                <div class="validation-error">你漏寫資料，請檢查和填寫所有資料</div>
                            </div>
                            <div>
                                <label class="block mb-1 font-semibold">支付方式</label>
                                <select id="paymentMethod" class="input-box" onchange="togglePaymentOther(); validateContractDetails(this);">
                                    <option value="">請選擇</option>
                                    <option value="In Cash">現金 In Cash</option>
                                    <option value="By Cheque">支票 By Cheque</option>
                                    <option value="Bank Transfer">銀行轉賬 Bank Transfer</option>
                                    <option value="Other">其他 Other</option>
                                </select>
                                <div class="validation-error">你漏寫資料，請檢查和填寫所有資料</div>

                                <input type="text" id="paymentMethodOther" class="input-box mt-2 hidden" placeholder="請以英文填寫支付方式" onblur="validateContractDetails(this)">
                                <div class="validation-error">你漏寫資料，請檢查和填寫所有資料</div>
                            </div>
                        </div>
                    </div>

                    <!-- Action Button -->
                    <button type="button" onclick="goToPreview()" class="w-full bg-royal-blue text-white font-bold py-4 rounded text-xl shadow hover:opacity-90 transition flex items-center justify-center gap-3">
                        <i class="fa-solid fa-calculator"></i> 計算銀碼
                    </button>

                </form>
            </div>

            <!-- PART 2: PREVIEW RESULT -->
            <div id="previewResultSection" class="hidden">
                <div class="flex justify-between items-center max-w-4xl mx-auto mb-4">
                    <button onclick="backToEdit()" class="bg-gray-500 text-white px-4 py-2 rounded hover:bg-gray-600"><i class="fa-solid fa-arrow-left"></i> 返回修改</button>
                    <button onclick="exportPDF()" class="bg-red-600 text-white px-6 py-2 rounded hover:bg-red-700 font-bold"><i class="fa-solid fa-file-pdf"></i> 輸出 PDF</button>
                </div>

                <div class="bg-light-yellow text-black text-center py-2 mb-4 font-bold border border-gray-300">
                    預覽計算結果 (可以直接點擊金額修改)
                </div>

                <div id="pdfContent" class="preview-page shadow-2xl mx-auto">
                    <div class="text-center mb-6">
                        <h1 class="text-xl font-bold uppercase">在僱傭合約終止／屆滿時所付款項的收據</h1>
                        <h2 class="text-lg font-bold">Receipt for Payments Upon Termination／Expiry of Employment Contract</h2>
                    </div>

                    <!-- Header Details Vertical -->
                    <div class="mb-6 text-sm leading-8">
                        <div class="flex justify-between">
                            <span>僱主姓名 Employer Name：</span>
                            <span id="p-employerName" class="font-mono border-b border-black px-2 flex-grow text-right min-w-[200px]"></span>
                        </div>
                        <div class="flex justify-between">
                            <span>外傭姓名 FDH Name：</span>
                            <span id="p-helperName" class="font-mono border-b border-black px-2 flex-grow text-right min-w-[200px]"></span>
                        </div>
                        <div class="flex justify-between">
                            <span>外傭香港身份證號碼／護照號碼<br>FDH HKID No.／Passport No.：</span>
                            <span id="p-helperId" class="font-mono border-b border-black px-2 flex-grow text-right self-end min-w-[200px]"></span>
                        </div>
                        <div class="flex justify-between">
                            <span>合約號碼 Contract No.：</span>
                            <span id="p-contractNo" class="font-mono border-b border-black px-2 flex-grow text-right min-w-[200px]"></span>
                        </div>
                        <div class="flex justify-between">
                            <span>款項收妥日期 Payment Received Date：</span>
                            <span id="p-paymentDate" class="font-mono border-b border-black px-2 flex-grow text-right min-w-[200px]"></span>
                        </div>
                        <div class="flex justify-between">
                            <span>支付方式 Payment Method：</span>
                            <span id="p-paymentMethod" class="font-mono border-b border-black px-2 flex-grow text-right min-w-[200px]"></span>
                        </div>
                    </div>

                    <!-- Items Table -->
                    <div class="space-y-4 text-sm">
                        <!-- Item 1 -->
                        <div class="flex justify-between items-end">
                            <div class="w-3/4">
                                <div class="font-bold">(1) 最後工資 Last wages</div>
                                <div class="pl-4">由 from <span id="p-lastWageStart"></span> 至 to <span id="p-lastWageEnd"></span></div>
                            </div>
                            <div class="w-1/4 text-right border-b border-black">
                                <span class="preview-currency mr-1">HK$</span> <span contenteditable="true" id="p-amount-1" class="preview-amount">0.0</span>
                            </div>
                        </div>

                        <!-- Item 2 -->
                        <div class="flex justify-between items-end">
                            <div class="w-3/4">
                                <div class="font-bold">(2) 未放取假期薪酬 Untaken leave pay</div>
                                <div class="pl-4">未放年假+假期:<span id="p-untakenDays"></span>日 Untaken paid annual leave+Untaken holiday:<span id="p-untakenDaysEn"></span> day</div>
                            </div>
                            <div class="w-1/4 text-right border-b border-black">
                                <span class="preview-currency mr-1">HK$</span> <span contenteditable="true" id="p-amount-2" class="preview-amount">0.0</span>
                            </div>
                        </div>

                        <!-- Item 3 -->
                        <div class="flex justify-between items-end">
                            <div class="w-3/4">
                                <div class="font-bold">(3) 交通津貼 Travelling allowance</div>
                            </div>
                            <div class="w-1/4 text-right border-b border-black">
                                <span class="preview-currency mr-1">HK$</span> <span contenteditable="true" id="p-amount-3" class="preview-amount">0.0</span>
                            </div>
                        </div>

                        <!-- Item 4 -->
                        <div class="flex justify-between items-end">
                            <div class="w-3/4">
                                <div class="font-bold">(4) 回國機票款項 Payment in lieu of return air ticket</div>
                            </div>
                            <div class="w-1/4 text-right border-b border-black">
                                <span id="p-amount-4-prefix" class="preview-currency mr-1">HK$</span> <span contenteditable="true" id="p-amount-4" class="preview-amount">0.0</span>
                            </div>
                        </div>

                        <!-- Item 5 -->
                        <div class="flex justify-between items-end">
                            <div class="w-3/4">
                                <div class="font-bold">(5) 代通知金 (如有) Payment in lieu of notice (if any)</div>
                                <div id="p-noticeDateDisplay" class="pl-4 hidden text-gray-600"></div>
                            </div>
                            <div class="w-1/4 text-right border-b border-black">
                                <span id="p-amount-5-prefix" class="preview-currency mr-1">HK$</span> <span contenteditable="true" id="p-amount-5" class="preview-amount">0.0</span>
                            </div>
                        </div>

                        <!-- Item 6 -->
                        <div class="flex justify-between items-end">
                            <div class="w-3/4">
                                <div class="font-bold">(6) 長期服務金 (如有) Long service payment (if any)</div>
                            </div>
                            <div class="w-1/4 text-right border-b border-black">
                                <span class="preview-currency mr-1">HK$</span> <span contenteditable="true" id="p-amount-6" class="preview-amount">0.0</span>
                            </div>
                        </div>

                        <!-- Item 7 -->
                        <div class="flex justify-between items-end">
                            <div class="w-3/4">
                                <div class="font-bold">(7) 其他 (如有) Other (if any)</div>
                            </div>
                            <div class="w-1/4 text-right border-b border-black">
                                <span class="preview-currency mr-1">HK$</span> <span contenteditable="true" id="p-amount-7" class="preview-amount">0.0</span>
                            </div>
                        </div>

                        <!-- Total -->
                        <div class="flex justify-between items-end pt-2">
                            <div class="w-3/4">
                                <div class="font-bold text-lg">(8) 合共金額 TOTAL AMOUNT</div>
                            </div>
                            <div class="w-1/4 text-right border-b-2 border-black font-bold text-lg">
                                <span class="preview-currency mr-1">HK$</span> <span contenteditable="true" id="p-total" class="preview-amount">0.0</span>
                            </div>
                        </div>

                        <!-- Remark -->
                        <div class="mt-4">
                            <div class="font-bold">(9) 備註 (如有) Remark：</div>
                            <!-- Adjusted margin for line moving down 5px and added pb-2 for line distance -->
                            <div class="border-b border-black min-h-[1.5em] pl-2 mt-[5px] pb-2" id="p-remark"></div>
                        </div>
                    </div>

                    <!-- Declaration -->
                    <div class="mt-6 text-xs text-justify leading-tight">
                        <p class="mb-2 font-bold">Declaration: We (the employer and the FDH) have agreed to and accepted all payments, holidays, and air-ticket arrangements. We will not make any form of complaint to the Philippine Consulate, Indonesian Consulate, Labour Department, Immigration Department, police station, and/or other government departments, nor will we seek any further losses, responsibilities, and/or compensation from the Hong Kong agency now or in the future.</p>
                        <p>聲明：我們（僱主及外傭）已同意及接受所有款項、假期和機票安排，並且我們不會向菲律賓領事館、印尼領事館、勞工處、入境處、警署及／或其他政府部門以作任何形式的投訴香港代理及／或日後追討其他損失、責任及／或款項。</p>
                        <p class="mt-2 italic text-center">(簽名式樣必須與合約簽名相同。Signature should agree with that on the contract.)</p>
                    </div>

                    <!-- Signatures -->
                    <!-- Added mt-5 to the container to move the lines down (increase space above them) -->
                    <div class="mt-[60px] flex justify-between">
                        <div class="text-center w-5/12">
                            <div class="border-t border-black pt-2">僱主簽署 Signature of Employer</div>
                        </div>
                        <div class="text-center w-5/12">
                            <div class="border-t border-black pt-2">外傭簽署 Signature of FDH</div>
                        </div>
                    </div>
                </div>

                <div class="text-center mt-6">
                    <button onclick="exportPDF()" class="bg-red-600 text-white px-8 py-3 rounded-lg hover:bg-red-700 font-bold text-lg shadow-lg">
                        <i class="fa-solid fa-download mr-2"></i> 立即輸出 PDF
                    </button>
                </div>
            </div>
        </div>


        <!-- ========================================== -->
        <!-- TAB 2: 4/6/8 WEEKS CALCULATOR -->
        <!-- ========================================== -->
        <div id="tab-weeks" class="tab-content hidden text-enlarged">
            <div class="bg-white p-6 rounded-lg shadow-lg max-w-2xl mx-auto">
                <div class="mb-6">
                    <label class="block font-semibold mb-2">📅 簽證完約日期 或 死移財最後工作日期</label>
                    <input type="date" id="visaEndDate" class="input-box" onchange="calcWeeks()">
                </div>

                <!-- Increased line-height by 3px approx (28+3 = 31px) -->
                <div class="mb-4 bg-light-yellow p-3 rounded border border-yellow-200" style="line-height: 31px;">
                    <p class="text-sm font-semibold text-plus-5">如果是印尼外傭，請在「最早可遞交入境處」前2星期就開始驗身和遞交文件給印尼領事館</p>
                </div>

                <div class="space-y-4">
                    <div>
                        <label class="block font-semibold text-gray-600 text-sm">完約轉換僱主 或 死移財轉換僱主 (4週)</label>
                        <div id="res-4weeks" class="calc-result-box font-mono font-bold">-</div>
                        <div class="text-xs text-black text-left">最早可遞交入境處</div>
                    </div>

                    <div>
                        <label class="block font-semibold text-gray-600 text-sm">Early Release 的最早走日期 (6週)</label>
                        <div id="res-6weeks" class="calc-result-box font-mono font-bold">-</div>
                        <div class="text-xs text-black text-left">此日期或之後可視作完約早走Early Release，此日期之前就視作斷約</div>
                    </div>

                    <div>
                        <label class="block font-semibold text-gray-600 text-sm">Early Release 的交入境處日期</label>
                        <div id="res-earliest" class="calc-result-box font-mono font-bold">-</div>
                        <div class="text-xs text-black text-left">最早可遞交入境處</div>
                    </div>

                    <div>
                        <label class="block font-semibold text-gray-600 text-sm">續約 (8週)</label>
                        <div id="res-8weeks" class="calc-result-box font-mono font-bold">-</div>
                        <div class="text-xs text-black text-left">最早可遞交入境處</div>
                    </div>
                </div>
            </div>
        </div>


        <!-- ========================================== -->
        <!-- TAB 3: LEAVE CALCULATOR -->
        <!-- ========================================== -->
        <div id="tab-leave" class="tab-content hidden space-y-8 text-enlarged">
            
            <!-- Forward -->
            <div class="bg-white p-6 rounded-lg shadow-lg max-w-3xl mx-auto border-t-4 border-blue-500">
                <div class="flex justify-between items-start mb-4">
                    <h3 class="text-xl font-bold">🗓️➡️ 順計日子 <span class="text-sm font-normal text-gray-600 block">(已有放假第一天，要計算最後一天)</span></h3>
                    <button onclick="clearSection('fwd')" class="text-sm bg-gray-200 hover:bg-gray-300 px-3 py-1 rounded btn-clear-large"><i class="fa-solid fa-eraser"></i> 清除內容</button>
                </div>
                
                <div class="space-y-4">
                    <div>
                        <label class="block text-sm font-semibold mb-1">放假第一天 (A)</label>
                        <input type="date" id="fwd-start" class="input-box" onchange="calcLeaveFwd()">
                    </div>
                    <div>
                        <label class="block text-sm font-semibold mb-1">放幾多日有薪年假 (想放足)</label>
                        <input type="number" id="fwd-days" class="input-box" onkeyup="calcLeaveFwd()">
                    </div>
                    <div>
                        <label class="block text-sm font-semibold mb-1">放假最後一天 (B) (小計)</label>
                        <div id="fwd-end-sub" class="calc-result-box bg-light-yellow">-</div>
                    </div>
                    <div>
                        <label class="block text-sm font-semibold mb-1">(A) 至 (B) 之間有多少個 休息日 和 勞工假期</label>
                        <input type="number" id="fwd-holidays" class="input-box" onkeyup="calcLeaveFwd()">
                    </div>
                    <div>
                        <label class="block text-sm font-semibold mb-1">放假最後一天 (最終)</label>
                        <div id="fwd-end-final" class="calc-result-box bg-light-yellow font-bold">-</div>
                    </div>
                    <div>
                        <label class="block text-sm font-semibold mb-1">橫跨日數</label>
                        <div id="fwd-span" class="calc-result-box bg-light-yellow">-</div>
                    </div>
                </div>
            </div>

            <!-- Backward -->
            <div class="bg-white p-6 rounded-lg shadow-lg max-w-3xl mx-auto border-t-4 border-green-500">
                <div class="flex justify-between items-start mb-4">
                    <h3 class="text-xl font-bold">🗓️⬅️ 倒計日子 <span class="text-sm font-normal text-gray-600 block">(已有放假最後一天，要計算第一天)</span></h3>
                    <button onclick="clearSection('bwd')" class="text-sm bg-gray-200 hover:bg-gray-300 px-3 py-1 rounded btn-clear-large"><i class="fa-solid fa-eraser"></i> 清除內容</button>
                </div>

                <div class="space-y-4">
                    <div>
                        <label class="block text-sm font-semibold mb-1">放假最後一天 (A)</label>
                        <input type="date" id="bwd-end" class="input-box" onchange="calcLeaveBwd()">
                    </div>
                    <div>
                        <label class="block text-sm font-semibold mb-1">放幾多日有薪年假 (想放足)</label>
                        <input type="number" id="bwd-days" class="input-box" onkeyup="calcLeaveBwd()">
                    </div>
                    <div>
                        <label class="block text-sm font-semibold mb-1">放假第一天 (B) (小計)</label>
                        <div id="bwd-start-sub" class="calc-result-box bg-light-yellow">-</div>
                    </div>
                    <div>
                        <label class="block text-sm font-semibold mb-1">(A) 至 (B) 之間有多少個 休息日 和 勞工假期</label>
                        <input type="number" id="bwd-holidays" class="input-box" onkeyup="calcLeaveBwd()">
                    </div>
                    <div>
                        <label class="block text-sm font-semibold mb-1">放假第一天 (最終)</label>
                        <div id="bwd-start-final" class="calc-result-box bg-light-yellow font-bold">-</div>
                    </div>
                    <div>
                        <label class="block text-sm font-semibold mb-1">橫跨日數</label>
                        <div id="bwd-span" class="calc-result-box bg-light-yellow">-</div>
                    </div>
                </div>
            </div>

            <!-- Range -->
            <div class="bg-white p-6 rounded-lg shadow-lg max-w-3xl mx-auto border-t-4 border-purple-500">
                <div class="flex justify-between items-start mb-4">
                    <h3 class="text-xl font-bold">🗓️↔️ 固定放假日子 <span class="text-sm font-normal text-gray-600 block">(不計算當中 休息日 和 勞工假期)</span></h3>
                    <button onclick="clearSection('rng')" class="text-sm bg-gray-200 hover:bg-gray-300 px-3 py-1 rounded btn-clear-large"><i class="fa-solid fa-eraser"></i> 清除內容</button>
                </div>

                <div class="space-y-4">
                    <div>
                        <label class="block text-sm font-semibold mb-1">放假第一天 (A) 最終</label>
                        <input type="date" id="rng-start" class="input-box" onchange="calcLeaveRng()">
                    </div>
                    <div>
                        <label class="block text-sm font-semibold mb-1">放假最後一天 (B) 最終</label>
                        <input type="date" id="rng-end" class="input-box" onchange="calcLeaveRng()">
                    </div>
                    <div>
                        <label class="block text-sm font-semibold mb-1">(A) 至 (B) 之間有多少個 休息日 和 勞工假期</label>
                        <input type="number" id="rng-holidays" class="input-box" onkeyup="calcLeaveRng()">
                    </div>
                    <div>
                        <label class="block text-sm font-semibold mb-1">實際放取有薪年假日數</label>
                        <div id="rng-actual" class="calc-result-box bg-light-yellow font-bold">-</div>
                    </div>
                    <div>
                        <label class="block text-sm font-semibold mb-1">橫跨日數</label>
                        <div id="rng-span" class="calc-result-box bg-light-yellow">-</div>
                    </div>
                </div>
            </div>

        </div>

    </div>

    <!-- JAVASCRIPT LOGIC -->
    <script>
        // === Navigation ===
        function switchTab(tabId) {
            document.querySelectorAll('.tab-content').forEach(el => el.classList.remove('block'));
            document.querySelectorAll('.tab-content').forEach(el => el.classList.add('hidden'));
            document.getElementById('tab-' + tabId).classList.remove('hidden');
            document.getElementById('tab-' + tabId).classList.add('block');

            document.querySelectorAll('.nav-btn').forEach(btn => btn.classList.remove('active'));
            document.getElementById('btn-' + tabId).classList.add('active');
        }

        // === Utility Functions ===
        const MS_PER_DAY = 1000 * 60 * 60 * 24;

        function addDays(dateStr, days) {
            if(!dateStr) return null;
            let d = new Date(dateStr);
            d.setDate(d.getDate() + parseInt(days));
            return d;
        }

        function dateDiff(startStr, endStr) {
            if(!startStr || !endStr) return 0;
            let d1 = new Date(startStr);
            let d2 = new Date(endStr);
            return (d2 - d1) / MS_PER_DAY;
        }

        function formatDate(dateObj) {
            if(!dateObj || isNaN(dateObj.getTime())) return "";
            let y = dateObj.getFullYear();
            let m = String(dateObj.getMonth() + 1).padStart(2, '0');
            let d = String(dateObj.getDate()).padStart(2, '0');
            return `${y}-${m}-${d}`;
        }
        
        function formatDisplayDate(dateStr) {
            if(!dateStr) return "";
            let [y, m, d] = dateStr.split('-');
            return `${d}-${m}-${y}`;
        }
        
        // Helper for dd-mm-yyyy only for specific sections
        function formatDateDDMMYYYY(dateObj) {
            if(!dateObj || isNaN(dateObj.getTime())) return "-";
            let y = dateObj.getFullYear();
            let m = String(dateObj.getMonth() + 1).padStart(2, '0');
            let d = String(dateObj.getDate()).padStart(2, '0');
            return `${d}-${m}-${y}`;
        }

        function truncate1Decimal(num) {
            return Math.floor(num * 10) / 10;
        }
        
        // Thousands separator
        function formatMoney(num) {
            if (isNaN(num)) return "0.0";
            let parts = truncate1Decimal(num).toFixed(1).split('.');
            parts[0] = parts[0].replace(/\B(?=(\d{3})+(?!\d))/g, ",");
            return parts.join('.');
        }

        // For display in input boxes with 3 decimal places
        function formatMoney3(num) {
            if (isNaN(num)) return "0.000";
            let parts = num.toFixed(3).split('.');
            parts[0] = parts[0].replace(/\B(?=(\d{3})+(?!\d))/g, ",");
            return parts.join('.');
        }

        // === Form Logic ===

        function toggleInput(id) {
            const el = document.getElementById(id);
            el.classList.toggle('hidden');
        }

        function toggleAirTicketInput() {
            const type = document.getElementById('airTicketType').value;
            const cashDiv = document.getElementById('airTicketCashInputDiv');
            if(type === '折現機票') {
                cashDiv.classList.remove('hidden');
            } else {
                cashDiv.classList.add('hidden');
                document.getElementById('airTicketCashValue').value = '';
                document.getElementById('airTicketCashValue').classList.remove('bg-red-50', 'border-red-500');
                const err = document.getElementById('airTicketCashValue').nextElementSibling;
                if(err) err.style.display = 'none';
            }
        }
        
        // Notice Input is always visible for all options now, just clearing error states
        function toggleNoticeInput() {
             // Just in case we need logic here
             document.getElementById('noticeDate').classList.remove('bg-red-50', 'border-red-500');
             const err = document.getElementById('noticeDate').nextElementSibling;
             if(err) err.style.display = 'none';
             document.getElementById('warn-notice-date').style.display = 'none';
        }

        function togglePaymentOther() {
            const val = document.getElementById('paymentMethod').value;
            const otherInput = document.getElementById('paymentMethodOther');
            if(val === 'Other') {
                otherInput.classList.remove('hidden');
            } else {
                otherInput.classList.add('hidden');
                otherInput.value = '';
            }
        }

        function clearTerminationForm() {
            const inputs = document.querySelectorAll('#termForm input, #termForm select');
            inputs.forEach(i => {
                if(i.type === 'checkbox') i.checked = false;
                else if(i.type === 'number' && (i.id === 'travelAllowance')) i.value = '100'; // Default
                else if(i.type === 'number') i.value = '';
                else if(i.type === 'date') i.value = '';
                else if(i.tagName === 'SELECT') i.selectedIndex = 0;
                else i.value = '';
            });
            // Reset displays
            document.querySelectorAll('#termForm .calc-result-box, #termForm .calc-result-box-plain').forEach(el => el.innerText = '-');
            document.getElementById('dailyRate').innerText = '0.000';
            document.getElementById('proRataLeave').innerText = '0.000';
            document.getElementById('finalLeaveDays').innerText = '0.000';
            // Notice input is not hidden anymore
            document.getElementById('lspInputGroup').classList.add('hidden');
            document.getElementById('detailsInputGroup').classList.add('hidden');
            document.getElementById('airTicketCashInputDiv').classList.add('hidden');
            document.querySelectorAll('.validation-error').forEach(e => e.style.display = 'none');
            document.getElementById('workYearsDisplay').classList.add('hidden');
        }

        function validateRequired(input) {
            const errDiv = input.nextElementSibling;
            if(!input.value) {
                input.classList.add('bg-red-50', 'border-red-500');
                if(errDiv && errDiv.classList.contains('validation-error')) errDiv.style.display = 'block';
            } else {
                input.classList.remove('bg-red-50', 'border-red-500');
                if(errDiv && errDiv.classList.contains('validation-error')) errDiv.style.display = 'none';
            }
        }

        function validateContractDetails(input) {
            // Check individual field if provided
            if(input) {
                 validateRequired(input);
            }
            // Logic for main submit button is handled in goToPreview
            return true; 
        }

        function validateSalary() {
            const input = document.getElementById('salary');
            const val = parseFloat(input.value);
            const errReq = document.getElementById('err-salary');
            const errLow = document.getElementById('warn-salary-low');
            
            if(!input.value) {
                errReq.style.display = 'block';
                input.classList.add('bg-red-50', 'border-red-500');
            } else {
                errReq.style.display = 'none';
                input.classList.remove('bg-red-50', 'border-red-500');
            }
            
            if(val < 4870) {
                errLow.style.display = 'block';
            } else {
                errLow.style.display = 'none';
            }
        }

        function checkThreeMonths() {
            const startStr = document.getElementById('lastContractStart').value;
            const endStr = document.getElementById('lastWorkDay').value;
            const warn = document.getElementById('warn-3months');
            
            if(startStr && endStr) {
                let dStart = new Date(startStr);
                let dEnd = new Date(endStr);
                
                // Add 3 months to start
                let d3Months = new Date(dStart);
                d3Months.setMonth(d3Months.getMonth() + 3);
                
                if (dEnd < d3Months) {
                    warn.style.display = 'block';
                } else {
                    warn.style.display = 'none';
                }
            } else {
                warn.style.display = 'none';
            }
        }

        function checkLatePayment() {
            const payEnd = document.getElementById('lastPayEnd').value;
            const workEnd = document.getElementById('lastWorkDay').value;
            const warn = document.getElementById('warn-latepay');
            
            // Logic: "離職月份的工作日數" > 35 show warning (Requested: 多過35)
            const workDaysEl = document.getElementById('workDaysResult');
            const days = parseInt(workDaysEl.innerText);
            
            if(!isNaN(days) && days > 35) warn.style.display = 'block';
            else warn.style.display = 'none';
        }

        function checkLSPValidity() {
            // Functionality moved to calcWorkYears logic for shared warning display
            calcWorkYears();
        }
        
        function checkNoticeDate() {
            const noticeDate = document.getElementById('noticeDate').value;
            const startStr = document.getElementById('periodStart').innerText; // Display text
            const endStr = document.getElementById('lastWorkDay').value; // Value
            const warn = document.getElementById('warn-notice-date');
            
            // PeriodStart text is display format dd-mm-yyyy, need to parse or use LastPayEnd + 1
            const lastPayEnd = document.getElementById('lastPayEnd').value;
            
            if(noticeDate && lastPayEnd && endStr) {
                let dNotice = new Date(noticeDate);
                let dStart = addDays(lastPayEnd, 1);
                let dEnd = new Date(endStr);
                
                // Allow notice date to be equal to start or end? "Between" usually inclusive.
                if(dNotice < dStart || dNotice > dEnd) {
                    warn.style.display = 'block';
                } else {
                    warn.style.display = 'none';
                }
            }
        }

        function calcWorkYears() {
            const first = document.getElementById('firstContractDate').value;
            const end = document.getElementById('lastWorkDay').value;
            const display = document.getElementById('workYearsDisplay');
            const warn = document.getElementById('warn-lsp-salary'); 
            
            if(first && end) {
                let days = dateDiff(first, end) + 1;
                let years = days / 365;
                display.innerText = `工作年期：${years.toFixed(2)}年 (共${days}日)`;
                display.classList.remove('hidden');
                display.classList.add('block');
                
                // Warning Logic: <= 1825 days
                if(days <= 1825) {
                    warn.style.display = 'block';
                } else {
                    warn.style.display = 'none';
                }
            } else {
                display.classList.add('hidden');
                warn.style.display = 'none';
            }
        }

        // --- Core Calculations ---

        function calcDailyRate() {
            const salary = parseFloat(document.getElementById('salary').value) || 0;
            const daily = (salary * 12) / 365;
            document.getElementById('dailyRate').innerText = formatMoney3(daily);
            return daily;
        }

        function calcPeriodStart() {
            const endStr = document.getElementById('lastPayEnd').value;
            if(endStr) {
                let d = addDays(endStr, 1);
                document.getElementById('periodStart').innerText = formatDisplayDate(formatDate(d));
                calcWorkDays();
            }
        }

        function calcWorkDays() {
            const endStr = document.getElementById('lastPayEnd').value;
            const lastWork = document.getElementById('lastWorkDay').value;
            
            if(endStr && lastWork) {
                let dStart = addDays(endStr, 1);
                let diff = dateDiff(formatDate(dStart), lastWork) + 1;
                document.getElementById('workDaysResult').innerText = diff;
                checkLatePayment(); // Check whenever days change
            }
        }

        function calcLeave() {
            const countVal = document.getElementById('contractCount').value;
            if(!countVal) {
                 document.getElementById('proRataLeave').innerText = '0.000';
                 calcFinalLeave();
                 return;
            }
            
            const count = parseInt(countVal);
            const startStr = document.getElementById('lastContractStart').value;
            const endStr = document.getElementById('lastWorkDay').value;
            
            if(!startStr || !endStr) return;

            let annualLeaveDays = 0;
            if(count === 1) annualLeaveDays = 14;
            else if(count === 2) annualLeaveDays = 17;
            else if(count === 3) annualLeaveDays = 21;
            else if(count === 4) annualLeaveDays = 25;
            else if(count >= 5) annualLeaveDays = 28;

            let diffDays = dateDiff(startStr, endStr) + 1;
            let proRata = (diffDays / 730) * annualLeaveDays;
            
            document.getElementById('proRataLeave').innerText = proRata.toFixed(3);
            calcFinalLeave();
        }

        function calcFinalLeave() {
            const proRata = parseFloat(document.getElementById('proRataLeave').innerText) || 0;
            const takenInput = document.getElementById('leaveTaken').value;
            const untakenInput = document.getElementById('holidaysUntaken').value;
            
            const taken = parseFloat(takenInput) || 0;
            const untakenHolidays = parseFloat(untakenInput) || 0;
            
            const final = proRata - taken + untakenHolidays;
            document.getElementById('finalLeaveDays').innerText = final.toFixed(3);
        }
        
        // --- Wages Logic Update ---
        function calcLastWagesAmount() {
             const salary = parseFloat(document.getElementById('salary').value) || 0;
             const dailyRate = parseFloat(document.getElementById('dailyRate').innerText.replace(/,/g,'')) || 0;
             const lastWorkStr = document.getElementById('lastWorkDay').value;
             const periodStartText = document.getElementById('periodStart').innerText; // Display format dd-mm-yyyy
             
             if(!lastWorkStr || periodStartText === '-') return 0;
             
             // Convert display text back to date object for comparison?
             // Or rely on lastPayEnd + 1
             const lastPayEnd = document.getElementById('lastPayEnd').value;
             let dPeriodStart = addDays(lastPayEnd, 1);
             let dLastWork = new Date(lastWorkStr);
             
             // 1. Check Full Month
             // Rule: LastWork + 1 - 1 Month.
             let dCheck = new Date(dLastWork);
             dCheck.setDate(dCheck.getDate() + 1);
             dCheck.setMonth(dCheck.getMonth() - 1);
             
             // Compare dCheck with dPeriodStart
             if(formatDate(dCheck) === formatDate(dPeriodStart)) {
                 return salary;
             }
             
             // 2. Check More than 1 Month
             if(dPeriodStart < dCheck) {
                 let dPrevMonthEnd = new Date(dLastWork.getFullYear(), dLastWork.getMonth(), 0); 
                 let diffDays = dateDiff(formatDate(dPrevMonthEnd), lastWorkStr); 
                 return (diffDays * dailyRate) + salary;
             }
             
             // 3. Partial Month (Default)
             const workDays = parseFloat(document.getElementById('workDaysResult').innerText);
             return workDays * dailyRate;
        }

        function goToPreview() {
            // Main Inputs Validation
            const inputs = document.querySelectorAll('#termForm input:not([type="checkbox"]), #termForm select');
            let isValid = true;
            
            inputs.forEach(input => {
                // Skips
                if(['remark', 'otherPayment', 'paymentMethodOther', 'airTicketCashValue'].includes(input.id)) return;
                if(input.id.startsWith('employer') || input.id.startsWith('helper') || input.id.startsWith('contract') || input.id.startsWith('payment')) return; // Details handled separately
                
                if(input.id === 'firstContractDate' && !document.getElementById('checkLSP').checked) return;

                // Explicitly valid empty or 0 inputs only if allowed (checked "leaveTaken" and "holidaysUntaken" are numeric and 0 is valid, but empty is not)
                // contractCount also cannot be empty
                if(!input.value && input.value !== 0 && input.value !== "0") {
                    isValid = false;
                    validateRequired(input);
                }
            });
            
            // Check Notice Pay Date
            // Always required for all options now
            const nDate = document.getElementById('noticeDate');
            if(!nDate.value) {
                isValid = false;
                validateRequired(nDate);
            }
            
            // Check Air Ticket Cash specific
            if(document.getElementById('airTicketType').value === '折現機票') {
                const cashInput = document.getElementById('airTicketCashValue');
                if(!cashInput.value) {
                    isValid = false;
                    validateRequired(cashInput);
                }
            }

            // Contract Details Validation (if checked)
            if(document.getElementById('checkDetails').checked) {
                const ids = ['employerName', 'helperName', 'helperId', 'contractNo', 'paymentDate', 'paymentMethod'];
                ids.forEach(id => {
                    const el = document.getElementById(id);
                    if(!el.value) isValid = false;
                    else validateRequired(el); 
                    
                    if(id === 'paymentMethod' && el.value === 'Other' && !document.getElementById('paymentMethodOther').value) {
                         isValid = false;
                         validateRequired(document.getElementById('paymentMethodOther'));
                    }
                });
            }

            if(!isValid) {
                alert("你漏寫資料，請檢查和填寫所有資料"); // Generic alert
                return;
            }

            transferToPreview();
            document.getElementById('inputFormSection').classList.add('hidden');
            document.getElementById('previewResultSection').classList.remove('hidden');
            window.scrollTo(0,0);
        }

        function backToEdit() {
            document.getElementById('inputFormSection').classList.remove('hidden');
            document.getElementById('previewResultSection').classList.add('hidden');
        }

        function transferToPreview() {
            // Details
            document.getElementById('p-employerName').innerText = document.getElementById('employerName').value || '';
            document.getElementById('p-helperName').innerText = document.getElementById('helperName').value || '';
            document.getElementById('p-helperId').innerText = document.getElementById('helperId').value || '';
            document.getElementById('p-contractNo').innerText = document.getElementById('contractNo').value || '';
            
            const pDate = document.getElementById('paymentDate').value;
            document.getElementById('p-paymentDate').innerText = pDate ? formatDisplayDate(pDate) : '';

            let pMethod = document.getElementById('paymentMethod').value;
            if(pMethod === 'Other') pMethod = document.getElementById('paymentMethodOther').value;
            document.getElementById('p-paymentMethod').innerText = pMethod || '';
            
            document.getElementById('p-remark').innerText = document.getElementById('remark').value;

            // (1) Last Wages
            const periodStart = document.getElementById('lastPayEnd').value ? addDays(document.getElementById('lastPayEnd').value, 1) : null;
            const lastDay = document.getElementById('lastWorkDay').value;
            
            document.getElementById('p-lastWageStart').innerText = periodStart ? formatDisplayDate(formatDate(periodStart)) : '';
            document.getElementById('p-lastWageEnd').innerText = formatDisplayDate(lastDay);
            
            let amount1 = calcLastWagesAmount();
            document.getElementById('p-amount-1').innerText = formatMoney(amount1);

            // (2) Leave Pay
            const finalLeave = parseFloat(document.getElementById('finalLeaveDays').innerText);
            document.getElementById('p-untakenDays').innerText = finalLeave.toFixed(1); 
            document.getElementById('p-untakenDaysEn').innerText = finalLeave.toFixed(1); 
            
            const dailyRate = parseFloat(document.getElementById('dailyRate').innerText.replace(/,/g,'')) || 0;
            let amount2 = finalLeave * dailyRate;
            document.getElementById('p-amount-2').innerText = formatMoney(amount2);

            // (3) Travel
            const travel = parseFloat(document.getElementById('travelAllowance').value) || 0;
            document.getElementById('p-amount-3').innerText = formatMoney(travel);

            // (4) Air Ticket
            const ticketType = document.getElementById('airTicketType').value;
            const amount4El = document.getElementById('p-amount-4');
            const prefix4 = document.getElementById('p-amount-4-prefix');
            
            if(ticketType === '折現機票') {
                let val = parseFloat(document.getElementById('airTicketCashValue').value) || 0;
                prefix4.classList.remove('hidden');
                amount4El.innerText = formatMoney(val);
            } else if (ticketType === '會買機票') {
                prefix4.classList.add('hidden');
                amount4El.innerText = "preparing to buy air ticket";
            } else {
                prefix4.classList.add('hidden');
                amount4El.innerText = "air ticket already bought";
            }

            // (5) Notice Pay
            let amount5 = 0;
            const noticeType = document.getElementById('noticePayType').value;
            const amount5El = document.getElementById('p-amount-5');
            const prefix5 = document.getElementById('p-amount-5-prefix');
            const noticeDisplay = document.getElementById('p-noticeDateDisplay');
            const noticeDate = document.getElementById('noticeDate').value;
            
            // Show informed date line
            if(noticeDate) {
                noticeDisplay.innerText = `通知日期 informed on ${formatDisplayDate(noticeDate)}`;
                noticeDisplay.classList.remove('hidden');
                noticeDisplay.classList.add('block');
            } else {
                noticeDisplay.classList.add('hidden');
            }
            
            // Default show prefix
            prefix5.classList.remove('hidden');

            if(noticeType === '給一個月通知期') {
                prefix5.classList.add('hidden');
                amount5El.innerText = "1 month notice";
            } else if(noticeType === '給一個月代通知金') {
                amount5 = parseFloat(document.getElementById('salary').value) || 0;
                amount5El.innerText = formatMoney(amount5);
            } else if(noticeType === '給部分代通知金') {
                const termDate = document.getElementById('lastWorkDay').value;
                if(noticeDate && termDate) {
                    let diff = dateDiff(noticeDate, termDate);
                    amount5 = diff * dailyRate;
                    amount5El.innerText = formatMoney(amount5);
                }
            }

            // (6) LSP
            let amount6 = 0;
            if(document.getElementById('checkLSP').checked) {
                const salary = parseFloat(document.getElementById('salary').value) || 0;
                const first = document.getElementById('firstContractDate').value;
                const end = document.getElementById('lastWorkDay').value;
                if(first && end) {
                    let years = (dateDiff(first, end) + 1) / 365;
                    amount6 = salary * (2/3) * years;
                }
            }
            document.getElementById('p-amount-6').innerText = formatMoney(amount6);

            // (7) Other
            const other = parseFloat(document.getElementById('otherPayment').value) || 0;
            document.getElementById('p-amount-7').innerText = formatMoney(other);

            // Total
            updateTotal();

            // Listeners
            for(let i=1; i<=7; i++) {
                document.getElementById(`p-amount-${i}`).addEventListener('input', updateTotal);
            }
        }

        function updateTotal() {
            let total = 0;
            const ids = [1, 2, 3, 6, 7]; // 5 handled separate logic below
            ids.forEach(i => {
                let txt = document.getElementById(`p-amount-${i}`).innerText.replace(/,/g, '');
                let val = parseFloat(txt) || 0;
                total += val;
            });

            // Item 4
            let txt4 = document.getElementById(`p-amount-4`).innerText;
            if(!isNaN(parseFloat(txt4)) && txt4.indexOf('ticket') === -1) {
                total += parseFloat(txt4.replace(/,/g, '')) || 0;
            }
            
            // Item 5 (Notice)
            let txt5 = document.getElementById(`p-amount-5`).innerText;
            // Only add if it's a number (not "1 month notice")
            if(!isNaN(parseFloat(txt5)) && txt5.indexOf('notice') === -1) {
                 total += parseFloat(txt5.replace(/,/g, '')) || 0;
            }

            document.getElementById('p-total').innerText = formatMoney(total);
        }

        async function exportPDF() {
            const { jsPDF } = window.jspdf;
            const element = document.getElementById('pdfContent');
            const btn = document.querySelector('#previewResultSection button.bg-red-600');
            const originalText = btn.innerHTML;
            btn.innerHTML = '<i class="fa-solid fa-spinner fa-spin"></i> 處理中...';

            const canvas = await html2canvas(element, { scale: 2, useCORS: true });
            const imgData = canvas.toDataURL('image/png');
            const pdf = new jsPDF('p', 'mm', 'a4');
            const pdfWidth = pdf.internal.pageSize.getWidth();
            const pdfHeight = (canvas.height * pdfWidth) / canvas.width;

            pdf.addImage(imgData, 'PNG', 0, 0, pdfWidth, pdfHeight);
            
            // Dynamic Filename
            let cNo = document.getElementById('contractNo').value || 'Contract';
            pdf.save(`${cNo}_Final_Payment.pdf`);

            btn.innerHTML = originalText;
        }

        // === Weeks Calculator ===
        function calcWeeks() {
            const input = document.getElementById('visaEndDate').value;
            if(!input) return;

            // Earliest Submit = -28
            let d28 = addDays(input, -28);
            document.getElementById('res-4weeks').innerText = formatDateDDMMYYYY(d28);

            // Early Release Date (6 weeks) = -42
            let d42 = addDays(input, -42);
            document.getElementById('res-6weeks').innerText = formatDateDDMMYYYY(d42);

            // Early Release Submit = Early Release Date - 28 = -70
            let dEarlySubmit = addDays(formatDate(d42), -28);
            document.getElementById('res-earliest').innerText = formatDateDDMMYYYY(dEarlySubmit);

            // Renewal Submit = -56
            let d56 = addDays(input, -56);
            document.getElementById('res-8weeks').innerText = formatDateDDMMYYYY(d56);
        }

        // === Leave Calculator ===
        
        function clearSection(prefix) {
            const inputs = document.querySelectorAll(`#tab-leave input[id^="${prefix}"]`);
            inputs.forEach(i => {
                if(i.type === 'number') i.value = ''; // Use empty string to force "missing" state
                else i.value = '';
            });
            
            // Clear results
            document.querySelectorAll(`#tab-leave .calc-result-box[id^="${prefix}"]`).forEach(el => {
                el.innerText = '-';
            });
        }

        // FWD
        function calcLeaveFwd() {
            const start = document.getElementById('fwd-start').value;
            const daysStr = document.getElementById('fwd-days').value;
            const holsStr = document.getElementById('fwd-holidays').value;

            // Only calc sub total if start & days present
            if(start && daysStr !== '') {
                const days = parseInt(daysStr);
                let dSub = addDays(start, days - 1);
                document.getElementById('fwd-end-sub').innerText = formatDateDDMMYYYY(dSub);
                
                // Only calc final if hols present
                if(holsStr !== '') {
                    const hols = parseInt(holsStr);
                    let dFinal = addDays(formatDate(dSub), hols);
                    document.getElementById('fwd-end-final').innerText = formatDateDDMMYYYY(dFinal);

                    let span = dateDiff(start, formatDate(dFinal)) + 1;
                    document.getElementById('fwd-span').innerText = span + " 日";
                } else {
                     document.getElementById('fwd-end-final').innerText = '-';
                     document.getElementById('fwd-span').innerText = '-';
                }
            } else {
                 document.getElementById('fwd-end-sub').innerText = '-';
                 document.getElementById('fwd-end-final').innerText = '-';
                 document.getElementById('fwd-span').innerText = '-';
            }
        }

        // BWD
        function calcLeaveBwd() {
            const end = document.getElementById('bwd-end').value;
            const daysStr = document.getElementById('bwd-days').value;
            const holsStr = document.getElementById('bwd-holidays').value;

            if(end && daysStr !== '') {
                const days = parseInt(daysStr);
                let dSub = addDays(end, -(days - 1));
                document.getElementById('bwd-start-sub').innerText = formatDateDDMMYYYY(dSub);

                if(holsStr !== '') {
                    const hols = parseInt(holsStr);
                    let dFinal = addDays(formatDate(dSub), -hols);
                    document.getElementById('bwd-start-final').innerText = formatDateDDMMYYYY(dFinal);

                    let span = dateDiff(formatDate(dFinal), end) + 1;
                    document.getElementById('bwd-span').innerText = span + " 日";
                } else {
                    document.getElementById('bwd-start-final').innerText = '-';
                    document.getElementById('bwd-span').innerText = '-';
                }
            } else {
                 document.getElementById('bwd-start-sub').innerText = '-';
                 document.getElementById('bwd-start-final').innerText = '-';
                 document.getElementById('bwd-span').innerText = '-';
            }
        }

        // Range
        function calcLeaveRng() {
            const start = document.getElementById('rng-start').value;
            const end = document.getElementById('rng-end').value;
            const holsStr = document.getElementById('rng-holidays').value;

            if(start && end && holsStr !== '') {
                const hols = parseInt(holsStr);
                let totalDays = dateDiff(start, end) + 1;
                let actual = totalDays - hols;
                
                document.getElementById('rng-actual').innerText = actual + " 日";
                document.getElementById('rng-span').innerText = totalDays + " 日";
            } else {
                document.getElementById('rng-actual').innerText = '-';
                document.getElementById('rng-span').innerText = '-';
            }
        }

    </script>
</body>
</html>
