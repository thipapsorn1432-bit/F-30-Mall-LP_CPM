<!DOCTYPE html>
<html lang="th">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Safety Patrol QR Scanner (46 จุดตรวจ)</title>
    
    <!-- Tailwind CSS for styling -->
    <script src="https://cdn.tailwindcss.com"></script>
    
    <!-- FontAwesome for Icons -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css" />
    
    <!-- React & ReactDOM -->
    <script crossorigin src="https://unpkg.com/react@18/umd/react.production.min.js"></script>
    <script crossorigin src="https://unpkg.com/react-dom@18/umd/react-dom.production.min.js"></script>
    
    <!-- Babel for in-browser JSX compilation -->
    <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>

    <style>
        /* Printable PDF A4 Formatting Diffs */
        @media print {
            body { background-color: #ffffff !important; color: #000000 !important; margin: 0 !important; padding: 0 !important; -webkit-print-color-adjust: exact !important; print-color-adjust: exact !important; }
            header, section, .no-print, nav, button, input, select, .modal-backdrop { display: none !important; }
            .print-area { display: block !important; width: 100% !important; margin: 0 !important; padding: 8mm !important; }
            .page-break { page-break-after: always; }
            table { border-collapse: collapse !important; width: 100% !important; }
            th, td { border: 1px solid #64748b !important; padding: 4px 6px !important; font-size: 9pt !important; }
        }
    </style>
</head>
<body class="bg-slate-50 text-slate-800 font-sans">
    <div id="root"></div>

    <script type="text/babel">
        const { useState, useMemo, useRef, useEffect } = React;

        // External jsQR loader script component helper
        const loadJsQR = () => {
            return new Promise((resolve) => {
                if (window.jsQR) { resolve(window.jsQR); return; }
                const script = document.createElement('script');
                script.src = 'https://cdn.jsdelivr.net/npm/jsqr@1.4.0/dist/jsQR.js';
                script.onload = () => resolve(window.jsQR);
                script.onerror = () => resolve(null);
                document.head.appendChild(script);
            });
        };

        // Load QRCode.js once.
        let qrCodeLibraryPromise;
        const loadQRCodeLibrary = () => {
            if (window.QRCode) return Promise.resolve(window.QRCode);
            if (!qrCodeLibraryPromise) {
                qrCodeLibraryPromise = new Promise((resolve, reject) => {
                    const script = document.createElement('script');
                    script.src = 'https://cdnjs.cloudflare.com/ajax/libs/qrcodejs/1.0.0/qrcode.min.js';
                    script.async = true;
                    script.onload = () => window.QRCode ? resolve(window.QRCode) : reject(new Error('โหลดไลบรารี QR Code ไม่สำเร็จ'));
                    script.onerror = () => reject(new Error('ไม่สามารถเชื่อมต่อไลบรารี QR Code ได้'));
                    document.head.appendChild(script);
                });
            }
            return qrCodeLibraryPromise;
        };

        function QRCodeLabel({ point }) {
            const qrRef = useRef(null);
            const [qrError, setQrError] = useState('');

            useEffect(() => {
                let cancelled = false;
                loadQRCodeLibrary()
                .then((QRCodeLib) => {
                    if (cancelled || !qrRef.current) return;
                    qrRef.current.innerHTML = '';
                    new QRCodeLib(qrRef.current, {
                        text: point.code,
                        width: 176,
                        height: 176,
                        colorDark: '#0f172a',
                        colorLight: '#ffffff',
                        correctLevel: QRCodeLib.CorrectLevel.M
                    });
                    setQrError('');
                })
                .catch((error) => {
                    if (!cancelled) setQrError(error.message || 'สร้าง QR Code ไม่สำเร็จ');
                });
                return () => { cancelled = true; };
            }, [point.code]);

            return (
                <div className="min-h-[176px] min-w-[176px] p-2 bg-white rounded-lg flex items-center justify-center">
                    <div ref={qrRef} aria-label={`QR ${point.code}`} />
                    {qrError && <span className="text-xs text-rose-600">{qrError}</span>}
                </div>
            );
        }

        // Data structure for 46 Checkpoints across 3 main zones
        const CHECKPOINT_ZONES = [
            {
                id: 'zone_1f', name: '1 FLOOR (15 จุดตรวจ)', icon: 'fa-solid fa-building',
                points: [
                    { id: 1, title: 'FHC Loading', code: 'SAFETY-CHK-01', category: '1f' },
                    { id: 2, title: 'ประตูหนีไฟแคนทีน', code: 'SAFETY-CHK-02', category: '1f' },
                    { id: 3, title: 'EE ลิฟต์ Staff', code: 'SAFETY-CHK-03', category: '1f' },
                    { id: 4, title: 'AHU Stock Diff', code: 'SAFETY-CHK-04', category: '1f' },
                    { id: 5, title: 'หน้าห้องการเงิน', code: 'SAFETY-CHK-05', category: '1f' },
                    { id: 6, title: 'หลังตู้ Cosmetic Fa (Jubilee)', code: 'SAFETY-CHK-06', category: '1f' },
                    { id: 7, title: 'EE-AHU Stock Men\'s', code: 'SAFETY-CHK-07', category: '1f' },
                    { id: 8, title: 'ATM', code: 'SAFETY-CHK-08', category: '1f' },
                    { id: 9, title: 'ประตู Auto door Power Buy', code: 'SAFETY-CHK-09', category: '1f' },
                    { id: 10, title: 'ลานโปรโมชั่น 1 (KFC)', code: 'SAFETY-CHK-10', category: '1f' },
                    { id: 11, title: 'ประตู Auto door KFC', code: 'SAFETY-CHK-11', category: '1f' },
                    { id: 12, title: 'FHC ลานโปรโมชั่น (TOPS)', code: 'SAFETY-CHK-12', category: '1f' },
                    { id: 13, title: 'ประตู Auto Door TOPS', code: 'SAFETY-CHK-13', category: '1f' },
                    { id: 14, title: 'FHC 2 TOPS', code: 'SAFETY-CHK-14', category: '1f' },
                    { id: 15, title: 'ห้องระบบ TOPS', code: 'SAFETY-CHK-15', category: '1f' }
                ]
            },
            {
                id: 'zone_2f', name: '2 FLOOR (21 จุดตรวจ)', icon: 'fa-solid fa-building',
                points: [
                    { id: 16, title: 'Generator Room', code: 'SAFETY-CHK-16', category: '2f' },
                    { id: 17, title: 'ประตูหนีไฟ Chiller', code: 'SAFETY-CHK-17', category: '2f' },
                    { id: 18, title: 'EE.หน้าห้อง I Care', code: 'SAFETY-CHK-18', category: '2f' },
                    { id: 19, title: 'AHU หน้าแคนทีน', code: 'SAFETY-CHK-19', category: '2f' },
                    { id: 20, title: 'หลังร้านแคนทีน (ห้องปรุง)', code: 'SAFETY-CHK-20', category: '2f' },
                    { id: 21, title: 'FHC Just Buy', code: 'SAFETY-CHK-21', category: '2f' },
                    { id: 22, title: 'FHC เครื่องครัว', code: 'SAFETY-CHK-22', category: '2f' },
                    { id: 23, title: 'AHU Stock Small App', code: 'SAFETY-CHK-23', category: '2f' },
                    { id: 24, title: 'EE.Stock Small App', code: 'SAFETY-CHK-24', category: '2f' },
                    { id: 25, title: 'FHC หน้าห้องน้ำ Food Park', code: 'SAFETY-CHK-25', category: '2f' },
                    { id: 26, title: 'Food Park', code: 'SAFETY-CHK-26', category: '2f' },
                    { id: 27, title: 'FHC The Cara Clinic', code: 'SAFETY-CHK-27', category: '2f' },
                    { id: 28, title: 'FHC True', code: 'SAFETY-CHK-28', category: '2f' },
                    { id: 29, title: 'ร้าน BANANA', code: 'SAFETY-CHK-29', category: '2f' },
                    { id: 30, title: 'ร้าน TRIPLE PHONE', code: 'SAFETY-CHK-30', category: '2f' },
                    { id: 31, title: 'FHC J-mart', code: 'SAFETY-CHK-31', category: '2f' },
                    { id: 32, title: 'FHC Playland', code: 'SAFETY-CHK-32', category: '2f' },
                    { id: 33, title: 'ทางเดินโรงภาพยนต์ 1', code: 'SAFETY-CHK-33', category: '2f' },
                    { id: 34, title: 'ทางเดินโรงภาพยนต์ 2', code: 'SAFETY-CHK-34', category: '2f' },
                    { id: 35, title: 'FHC SSP', code: 'SAFETY-CHK-35', category: '2f' },
                    { id: 36, title: 'ห้องน้ำลูกค้า Work Shop', code: 'SAFETY-CHK-36', category: '2f' }
                ]
            },
            {
                id: 'zone_outdoor', name: 'OUTDOOR (10 จุดตรวจ)', icon: 'fa-solid fa-tree',
                points: [
                    { id: 37, title: 'ห้องเก็บถังแก๊ส', code: 'SAFETY-CHK-37', category: 'outdoor' },
                    { id: 38, title: 'Fire Pump Room', code: 'SAFETY-CHK-38', category: 'outdoor' },
                    { id: 39, title: 'Reuse Room', code: 'SAFETY-CHK-39', category: 'outdoor' },
                    { id: 40, title: 'Tower Sing', code: 'SAFETY-CHK-40', category: 'outdoor' },
                    { id: 41, title: 'ป้าย SSP (ริมถนนเลี่ยงเมือง)', code: 'SAFETY-CHK-41', category: 'outdoor' },
                    { id: 42, title: 'ศาลพระพรหม', code: 'SAFETY-CHK-42', category: 'outdoor' },
                    { id: 43, title: 'เสากล้องโค้งท๊อปส์', code: 'SAFETY-CHK-43', category: 'outdoor' },
                    { id: 44, title: 'ประตูทางออก 1 (รพ.ชัยภูมิ-ราม)', code: 'SAFETY-CHK-44', category: 'outdoor' },
                    { id: 45, title: 'ตู้แดงลานจอดจุด VIP PARKING', code: 'SAFETY-CHK-45', category: 'outdoor' },
                    { id: 46, title: 'จุดตรวจภายนอกอาคารเพิ่มเติม', code: 'SAFETY-CHK-46', category: 'outdoor' }
                ]
            }
        ];

        function App() {
            const [activeTab, setActiveTab] = useState('inspect');

            const [inspectorInfo, setInspectorInfo] = useState({
                name: 'พรพรหม',
                department: 'แผนกป้องกันการสูญเสีย (LP)',
                date: new Date().toISOString().split('T')[0],
                shift: 'รอบที่ 1  14.00 น.  LP',
                notes: ''
            });

            const inspectorNames = [
                'พรพรหม', 'ณัฐกร', 'ประเสริฐ', 'อานนท์', 'ณัฐวุฒิ', 'อรัญ', 
                'ทิพย์อัปษร', 'ชนะชัย', 'ธิติพงษ์', 'ชัชวาล', 'นนทกร', 'ธนวัฒน์', 'วิษณุ', 'หัวหน้าชุด รปภ.'
            ];

            const [checklist, setChecklist] = useState(() => {
                const initial = {};
                CHECKPOINT_ZONES.forEach(zone => {
                    zone.points.forEach(p => {
                        initial[p.id] = { isScanned: false, scanTimestamp: null, status: 'pending', note: '', severity: 'Low' };
                    });
                });
                return initial;
            });

            const [searchTerm, setSearchTerm] = useState('');
            const [statusFilter, setStatusFilter] = useState('all');
            const [selectedZone, setSelectedZone] = useState('all');

            const [activeScanPoint, setActiveScanPoint] = useState(null);
            const [isGlobalScanMode, setIsGlobalScanMode] = useState(false);
            const [cameraActive, setCameraActive] = useState(false);
            const [cameraStarting, setCameraStarting] = useState(false);
            const [cameraError, setCameraError] = useState(null);
            const [scanMessage, setScanMessage] = useState('');

            const videoRef = useRef(null);
            const canvasRef = useRef(null);
            const streamRef = useRef(null);
            const animFrameRef = useRef(null);
            const scanBusyRef = useRef(false);

            const allPoints = useMemo(() => CHECKPOINT_ZONES.flatMap(z => z.points), []);

            const stats = useMemo(() => {
                let total = 46, normal = 0, issue = 0, pending = 0, scannedCount = 0;
                Object.values(checklist).forEach(item => {
                    if (item.isScanned) scannedCount++;
                    if (item.status === 'normal') normal++;
                    else if (item.status === 'issue') issue++;
                    else pending++;
                });
                return { total, normal, issue, pending, scannedCount };
            }, [checklist]);

            const triggerBeep = () => {
                try {
                    const audioCtx = new (window.AudioContext || window.webkitAudioContext)();
                    const osc = audioCtx.createOscillator();
                    const gain = audioCtx.createGain();
                    osc.type = 'sine';
                    osc.frequency.setValueAtTime(880, audioCtx.currentTime);
                    gain.gain.setValueAtTime(0.1, audioCtx.currentTime);
                    osc.connect(gain);
                    gain.connect(audioCtx.destination);
                    osc.start();
                    osc.stop(audioCtx.currentTime + 0.15);
                } catch (e) {}
            };

            const handleScanSuccess = (scannedText) => {
                const normalized = String(scannedText || '').trim().toUpperCase();
                const scannedPoint = allPoints.find(point => point.code.toUpperCase() === normalized);

                if (!scannedPoint) {
                    setCameraError('QR Code นี้ไม่ตรงกับรหัสจุดตรวจในระบบ');
                    return false;
                }
                if (checklist[scannedPoint.id]?.isScanned) {
                    setCameraError(`จุดตรวจ ${scannedPoint.title} ถูกสแกนไปแล้ว`);
                    return false;
                }
                if (activeScanPoint && scannedPoint.id !== activeScanPoint.id) {
                    setCameraError(`ขณะนี้เปิดสแกนสำหรับจุด ${activeScanPoint.id} เท่านั้น (พบ QR ของจุด ${scannedPoint.id})`);
                    return false;
                }

                if (scanBusyRef.current) return false;
                scanBusyRef.current = true;

                const now = new Date();
                const nowString = now.toLocaleTimeString('th-TH', { hour: '2-digit', minute: '2-digit', second: '2-digit' });

                setChecklist(prev => {
                    if (prev[scannedPoint.id]?.isScanned) return prev;
                    return {
                        ...prev,
                        [scannedPoint.id]: {
                            ...prev[scannedPoint.id],
                            isScanned: true,
                            scanTimestamp: nowString,
                            scanTimestampISO: now.toISOString(),
                            scannedCode: scannedPoint.code,
                            status: prev[scannedPoint.id].status === 'pending' ? 'normal' : prev[scannedPoint.id].status
                        }
                    };
                });

                triggerBeep();
                setScanMessage(`สแกนจุดตรวจ #${scannedPoint.id} (${scannedPoint.title}) สำเร็จ เวลา ${nowString} น.`);
                window.setTimeout(() => setScanMessage(''), 5000);

                stopCamera();
                setActiveScanPoint(null);
                setIsGlobalScanMode(false);
                return true;
            };

            const scanCanvas = (jsQR) => {
                if (!streamRef.current || !videoRef.current || !canvasRef.current || scanBusyRef.current) return;
                const video = videoRef.current;
                const canvas = canvasRef.current;
                const ctx = canvas.getContext('2d', { willReadFrequently: true });

                if (ctx && video.readyState >= 2 && video.videoWidth > 0 && video.videoHeight > 0) {
                    const maxSide = 900;
                    const scale = Math.min(1, maxSide / Math.max(video.videoWidth, video.videoHeight));
                    canvas.width = Math.round(video.videoWidth * scale);
                    canvas.height = Math.round(video.videoHeight * scale);
                    ctx.drawImage(video, 0, 0, canvas.width, canvas.height);

                    const imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);
                    const code = jsQR(imageData.data, imageData.width, imageData.height, { inversionAttempts: 'attemptBoth' });

                    if (code?.data) {
                        if (handleScanSuccess(code.data)) return;
                    }
                }

                if (streamRef.current) {
                    animFrameRef.current = requestAnimationFrame(() => scanCanvas(jsQR));
                }
            };

            const startCamera = async () => {
                if (streamRef.current || cameraStarting) return;
                setCameraStarting(true);
                setCameraError(null);
                scanBusyRef.current = false;

                if (!navigator.mediaDevices?.getUserMedia) {
                    setCameraError('เบราว์เซอร์นี้ไม่รองรับการเปิดกล้อง กรุณาเปิดลิงก์ผ่าน Safari (iPhone) หรือ Chrome (Android)');
                    setCameraStarting(false);
                    return;
                }

                let stream = null;
                let lastError = null;
                const cameraConstraints = [
                    { audio: false, video: { facingMode: { ideal: 'environment' } } },
                    { audio: false, video: true }
                ];

                try {
                    setCameraError('กำลังขออนุญาตใช้กล้อง...');
                    for (const constraints of cameraConstraints) {
                        try { stream = await navigator.mediaDevices.getUserMedia(constraints); break; }
                        catch (error) { lastError = error; }
                    }

                    if (!stream) throw lastError || new Error('ไม่สามารถเปิดกล้องได้');
                    streamRef.current = stream;

                    const video = videoRef.current;
                    if (!video) {
                        stream.getTracks().forEach(track => track.stop());
                        streamRef.current = null;
                        return;
                    }

                    video.setAttribute('playsinline', 'true');
                    video.setAttribute('webkit-playsinline', 'true');
                    video.muted = true;
                    video.autoplay = true;
                    video.srcObject = stream;
                    await video.play();

                    const jsQR = await loadJsQR();
                    if (!jsQR) throw new Error('โหลดระบบอ่าน QR Code ไม่สำเร็จ กรุณาตรวจสอบอินเทอร์เน็ตแล้วลองใหม่');

                    setCameraActive(true);
                    setCameraError('');
                    animFrameRef.current = requestAnimationFrame(() => scanCanvas(jsQR));
                } catch (error) {
                    if (streamRef.current) {
                        streamRef.current.getTracks().forEach(track => track.stop());
                        streamRef.current = null;
                    }
                    setCameraActive(false);
                    const messages = {
                        NotAllowedError: 'ไม่ได้รับอนุญาตให้ใช้กล้อง กรุณาอนุญาต Camera หรือเปิดลิงก์ผ่าน Safari / Chrome',
                        PermissionDeniedError: 'การใช้กล้องถูกปฏิเสธ กรุณาเปิดสิทธิ์กล้องในตั้งค่าเบราว์เซอร์',
                        NotFoundError: 'ไม่พบกล้องบนอุปกรณ์นี้',
                        NotReadableError: 'กล้องกำลังถูกใช้งานโดยแอปอื่น กรุณาปิดแอปกล้องแล้วลองใหม่'
                    };
                    setCameraError(messages[error?.name] || error?.message || 'ไม่สามารถเข้าถึงกล้องได้ กรุณาใช้ปุ่มอัปโหลดรูปภาพ');
                } finally {
                    setCameraStarting(false);
                }
            };

            const stopCamera = () => {
                if (animFrameRef.current) { cancelAnimationFrame(animFrameRef.current); animFrameRef.current = null; }
                if (streamRef.current) { streamRef.current.getTracks().forEach(track => track.stop()); streamRef.current = null; }
                if (videoRef.current) videoRef.current.srcObject = null;
                scanBusyRef.current = false;
                setCameraActive(false);
            };

            useEffect(() => {
                if (!activeScanPoint && !isGlobalScanMode) stopCamera();
                return () => stopCamera();
            }, [activeScanPoint, isGlobalScanMode]);

            const handleFileUploadScan = async (e) => {
                const file = e.target.files?.[0];
                if (!file) return;

                const jsQR = await loadJsQR();
                if (!jsQR) { setCameraError('ไม่สามารถโหลดตัวถอดรหัส QR ได้'); return; }

                const reader = new FileReader();
                reader.onload = (event) => {
                    const img = new Image();
                    img.onload = () => {
                        const canvas = document.createElement('canvas');
                        const ctx = canvas.getContext('2d');
                        canvas.width = img.width;
                        canvas.height = img.height;
                        ctx.drawImage(img, 0, 0);
                        const imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);
                        const code = jsQR(imageData.data, imageData.width, imageData.height, { inversionAttempts: 'attemptBoth' });

                        if (code?.data) {
                            handleScanSuccess(code.data);
                        } else {
                            setCameraError('ไม่พบ QR Code ในภาพที่อัปโหลด กรุณาลองใหม่อีกครั้ง');
                        }
                    };
                    img.src = event.target.result;
                };
                reader.readAsDataURL(file);
                e.target.value = '';
            };

            const handlePrintReport = () => window.print();

            const handleExportCSV = () => {
                let csvContent = "\uFEFF"; 
                csvContent += "ลำดับ,รหัสจุดตรวจ,ชื่อจุดตรวจ,โซน,สถานะ,ระดับความรุนแรง,เวลาสแกน QR,หมายเหตุ/ข้อเสนอแนะ,ผู้ตรวจ,วันที่,รอบการตรวจ\n";
                CHECKPOINT_ZONES.forEach(zone => {
                    zone.points.forEach(point => {
                        const item = checklist[point.id];
                        const statusText = item.status === 'normal' ? 'ปกติ' : item.status === 'issue' ? 'พบข้อบกพร่อง' : 'รอตรวจ';
                        const severityText = item.status === 'issue' ? item.severity : '-';
                        const timestamp = item.scanTimestamp || '-';
                        const noteText = item.note ? item.note.replace(/,/g, ' ') : '-';
                        csvContent += `${point.id},${point.code},"${point.title}","${zone.name}","${statusText}","${severityText}","${timestamp}","${noteText}","${inspectorInfo.name}","${inspectorInfo.date}","${inspectorInfo.shift}"\n`;
                    });
                });
                const blob = new Blob([csvContent], { type: 'text/csv;charset=utf-8;' });
                const url = URL.createObjectURL(blob);
                const link = document.createElement("a");
                link.setAttribute("href", url);
                link.setAttribute("download", `Safety_Patrol_${inspectorInfo.date}_${inspectorInfo.name.replace(/\s+/g, '_')}.csv`);
                document.body.appendChild(link);
                link.click();
                document.body.removeChild(link);
            };

            return (
                <div className="flex flex-col min-h-screen">
                    <header className="bg-slate-900 text-white sticky top-0 z-30 shadow-md no-print">
                        <div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
                            <div className="flex items-center justify-between h-16">
                                <div className="flex items-center space-x-3">
                                    <div className="bg-amber-500 text-slate-950 p-2 rounded-xl font-bold flex items-center justify-center shadow-md">
                                        <i className="fa-solid fa-shield-halved text-xl"></i>
                                    </div>
                                    <div>
                                        <h1 className="text-base sm:text-lg font-bold tracking-tight text-white flex items-center gap-2">
                                            Safety Patrol QR Scanner (46 จุด)
                                        </h1>
                                        <p className="text-xs text-slate-400 hidden sm:block">ระบบสแกนความปลอดภัย 3 โซน (1F, 2F, OUTDOOR)</p>
                                    </div>
                                </div>
                                <div className="flex items-center space-x-2">
                                    <div className="bg-slate-800 p-1 rounded-lg flex space-x-1 border border-slate-700">
                                        <button onClick={() => setActiveTab('inspect')} className={`px-3 py-1.5 rounded-md text-xs sm:text-sm font-medium transition-all flex items-center gap-1.5 ${activeTab === 'inspect' ? 'bg-amber-500 text-slate-950 font-bold shadow' : 'text-slate-300 hover:text-white'}`}>
                                            <i className="fa-solid fa-circle-check"></i>
                                            <span>เดินตรวจจุด</span>
                                        </button>
                                        <button onClick={() => setActiveTab('report')} className={`px-3 py-1.5 rounded-md text-xs sm:text-sm font-medium transition-all flex items-center gap-1.5 ${activeTab === 'report' ? 'bg-amber-500 text-slate-950 font-bold shadow' : 'text-slate-300 hover:text-white'}`}>
                                            <i className="fa-solid fa-file-lines"></i>
                                            <span>รายงานสรุป</span>
                                        </button>
                                        <button onClick={() => setActiveTab('qrgenerator')} className={`px-3 py-1.5 rounded-md text-xs sm:text-sm font-medium transition-all flex items-center gap-1.5 ${activeTab === 'qrgenerator' ? 'bg-amber-500 text-slate-950 font-bold shadow' : 'text-slate-300 hover:text-white'}`}>
                                            <i className="fa-solid fa-qrcode"></i>
                                            <span className="hidden md:inline">ป้าย QR Code</span>
                                        </button>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </header>

                    <section className="bg-white border-b border-slate-200 py-3.5 px-4 sm:px-6 lg:px-8 no-print">
                        <div className="max-w-7xl mx-auto">
                            <div className="grid grid-cols-1 md:grid-cols-3 gap-3 text-sm">
                                <div className="flex items-center space-x-3 bg-slate-50 p-2.5 rounded-lg border border-slate-200">
                                    <i className="fa-solid fa-user text-amber-600 flex-shrink-0"></i>
                                    <div className="flex-1">
                                        <label className="text-[11px] text-slate-500 font-medium block">ชื่อผู้ตรวจความปลอดภัย</label>
                                        <select value={inspectorInfo.name} onChange={(e) => setInspectorInfo({ ...inspectorInfo, name: e.target.value })} className="w-full bg-transparent font-semibold text-slate-800 border-none p-0 focus:ring-0 text-xs sm:text-sm cursor-pointer">
                                            {inspectorNames.map((name, index) => <option key={index} value={name}>{name}</option>)}
                                        </select>
                                    </div>
                                </div>
                                <div className="flex items-center space-x-3 bg-slate-50 p-2.5 rounded-lg border border-slate-200">
                                    <i className="fa-solid fa-calendar-days text-amber-600 flex-shrink-0"></i>
                                    <div className="flex-1">
                                        <label className="text-[11px] text-slate-500 font-medium block">วันที่ตรวจ</label>
                                        <input type="date" value={inspectorInfo.date} onChange={(e) => setInspectorInfo({ ...inspectorInfo, date: e.target.value })} className="w-full bg-transparent font-semibold text-slate-800 border-none p-0 focus:ring-0 text-xs sm:text-sm" />
                                    </div>
                                </div>
                                <div className="flex items-center space-x-3 bg-slate-50 p-2.5 rounded-lg border border-slate-200">
                                    <i className="fa-solid fa-clock text-amber-600 flex-shrink-0"></i>
                                    <div className="flex-1">
                                        <label className="text-[11px] text-slate-500 font-medium block">รอบการตรวจ</label>
                                        <select value={inspectorInfo.shift} onChange={(e) => setInspectorInfo({ ...inspectorInfo, shift: e.target.value })} className="w-full bg-transparent font-bold text-slate-900 border-none p-0 focus:ring-0 text-xs sm:text-sm cursor-pointer">
                                            <option value="รอบที่ 1  14.00 น.  LP">รอบที่ 1 14.00 น. LP</option>
                                            <option value="รอบที่ 2 17.00 น.  รปภ.">รอบที่ 2 17.00 น. รปภ.</option>
                                            <option value="รอบที่ 3 23.00 น. FM">รอบที่ 3 23.00 น. FM</option>
                                            <option value="รอบที่ 4 02.00 น. รปภ.">รอบที่ 4 02.00 น. รปภ.</option>
                                            <option value="รอบที่ 5 05.00 น. LP">รอบที่ 5 05.00 น. LP</option>
                                        </select>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </section>

                    <main className="flex-1 max-w-7xl w-full mx-auto px-4 sm:px-6 lg:px-8 py-6">
                        {scanMessage && (
                            <div className="mb-4 p-3 bg-emerald-600 text-white rounded-xl font-bold text-xs sm:text-sm shadow-md flex items-center justify-between animate-bounce no-print">
                                <div className="flex items-center gap-2">
                                    <i className="fa-solid fa-circle-check text-xl text-emerald-200"></i>
                                    <span>{scanMessage}</span>
                                </div>
                                <button onClick={() => setScanMessage('')} className="text-emerald-200 hover:text-white">
                                    <i className="fa-solid fa-xmark"></i>
                                </button>
                            </div>
                        )}

                        <div className="bg-white rounded-xl shadow-sm border border-slate-200 p-4 sm:p-5 mb-6 no-print">
                            <div className="flex flex-col lg:flex-row lg:items-center lg:justify-between gap-4 mb-4">
                                <div>
                                    <h2 className="text-base sm:text-lg font-bold text-slate-900 flex items-center gap-2">
                                        <i className="fa-solid fa-chart-column text-xl text-amber-500"></i>
                                        สรุปความคืบหน้าการเดินตรวจ (46 จุดตรวจ)
                                    </h2>
                                    <p className="text-xs text-slate-500">สแกน QR Code ผ่านกล้องหรืออัปโหลดรูปภาพ เพื่อปลดล็อกและบันทึกเวลา Real-time</p>
                                </div>
                                <div className="flex items-center space-x-3">
                                    <button onClick={() => { setCameraError(null); setIsGlobalScanMode(true); setActiveScanPoint(null); }} className="px-4 py-2 bg-amber-500 hover:bg-amber-400 text-slate-950 font-bold text-xs sm:text-sm rounded-lg shadow flex items-center gap-2">
                                        <i className="fa-solid fa-camera"></i>
                                        <span>เปิดกล้องสแกน QR (จุดใดก็ได้)</span>
                                    </button>
                                    <div className="hidden sm:flex items-center space-x-2">
                                        <div className="w-36 bg-slate-100 rounded-full h-3.5 overflow-hidden p-0.5 border border-slate-200">
                                            <div className={`h-full rounded-full transition-all duration-500 ${stats.scannedCount === 46 ? 'bg-emerald-500' : 'bg-cyan-500'}`} style={{ width: `${(stats.scannedCount / 46) * 100}%` }} />
                                        </div>
                                        <span className="text-xs font-bold text-slate-800 font-mono">{stats.scannedCount}/46</span>
                                    </div>
                                </div>
                            </div>
                            <div className="grid grid-cols-2 sm:grid-cols-5 gap-3">
                                <div className="p-3 rounded-xl bg-cyan-50 border border-cyan-200 text-cyan-950">
                                    <div className="text-xs font-semibold flex items-center gap-1 text-cyan-700">
                                        <i className="fa-solid fa-qrcode text-sm"></i><span>สแกน QR แล้ว</span>
                                    </div>
                                    <div className="text-2xl font-bold mt-1">{stats.scannedCount} <span className="text-xs font-normal">จุด</span></div>
                                </div>
                                <div onClick={() => setStatusFilter('all')} className={`p-3 rounded-xl border transition-all cursor-pointer ${statusFilter === 'all' ? 'bg-slate-900 text-white border-slate-900 shadow-md' : 'bg-slate-50 border-slate-200 hover:bg-slate-100'}`}>
                                    <div className="text-xs font-medium opacity-80">จุดตรวจทั้งหมด</div>
                                    <div className="text-2xl font-bold mt-1">{stats.total} <span className="text-xs font-normal">จุด</span></div>
                                </div>
                                <div onClick={() => setStatusFilter('normal')} className={`p-3 rounded-xl border transition-all cursor-pointer ${statusFilter === 'normal' ? 'bg-emerald-600 text-white border-emerald-600 shadow-md' : 'bg-emerald-50 border-emerald-200 hover:bg-emerald-100 text-emerald-900'}`}>
                                    <div className="text-xs font-medium flex items-center gap-1">
                                        <i className="fa-solid fa-circle-check text-emerald-500"></i><span>ปกติ</span>
                                    </div>
                                    <div className="text-2xl font-bold mt-1">{stats.normal} <span className="text-xs font-normal">จุด</span></div>
                                </div>
                                <div onClick={() => setStatusFilter('issue')} className={`p-3 rounded-xl border transition-all cursor-pointer ${statusFilter === 'issue' ? 'bg-rose-600 text-white border-rose-600 shadow-md' : 'bg-rose-50 border-rose-200 hover:bg-rose-100 text-rose-900'}`}>
                                    <div className="text-xs font-medium flex items-center gap-1">
                                        <i className="fa-solid fa-triangle-exclamation text-rose-500"></i><span>พบจุดแก้ไข</span>
                                    </div>
                                    <div className="text-2xl font-bold mt-1">{stats.issue} <span className="text-xs font-normal">จุด</span></div>
                                </div>
                                <div onClick={() => setStatusFilter('pending')} className={`p-3 rounded-xl border transition-all cursor-pointer ${statusFilter === 'pending' ? 'bg-amber-600 text-white border-amber-600 shadow-md' : 'bg-amber-50 border-amber-200 hover:bg-amber-100 text-amber-900'}`}>
                                    <div className="text-xs font-medium flex items-center gap-1">
                                        <i className="fa-solid fa-clock text-amber-500"></i><span>รอสแกน/ตรวจ</span>
                                    </div>
                                    <div className="text-2xl font-bold mt-1">{stats.pending} <span className="text-xs font-normal">จุด</span></div>
                                </div>
                            </div>
                        </div>

                        {activeTab === 'inspect' && (
                            <div className="space-y-6 no-print">
                                <div className="bg-white p-4 rounded-xl shadow-sm border border-slate-200 flex flex-col md:flex-row items-center justify-between gap-4">
                                    <div className="relative w-full md:w-72">
                                        <i className="fa-solid fa-magnifying-glass text-slate-400 absolute left-3 top-1/2 transform -translate-y-1/2"></i>
                                        <input type="text" placeholder="ค้นหาชื่อจุดตรวจ หรือ หมายเลข..." value={searchTerm} onChange={(e) => setSearchTerm(e.target.value)} className="w-full pl-9 pr-3 py-2 text-sm bg-slate-50 border border-slate-200 rounded-lg focus:outline-none focus:ring-2 focus:ring-amber-500" />
                                    </div>
                                    <div className="flex items-center space-x-2 w-full md:w-auto">
                                        <i className="fa-solid fa-filter text-slate-500 flex-shrink-0"></i>
                                        <select value={selectedZone} onChange={(e) => setSelectedZone(e.target.value)} className="w-full md:w-auto bg-slate-50 border border-slate-200 rounded-lg px-3 py-2 text-sm focus:outline-none focus:ring-2 focus:ring-amber-500 font-semibold">
                                            <option value="all">เลือกโซนทั้งหมด (3 โซนหลัก)</option>
                                            <option value="zone_1f">1 FLOOR (จุด 1 - 15)</option>
                                            <option value="zone_2f">2 FLOOR (จุด 16 - 36)</option>
                                            <option value="zone_outdoor">OUTDOOR (จุด 37 - 46)</option>
                                        </select>
                                    </div>
                                </div>

                                <div className="space-y-6">
                                    {CHECKPOINT_ZONES.map(zone => {
                                        if (selectedZone !== 'all' && selectedZone !== zone.id) return null;
                                        const filteredPoints = zone.points.filter(p => {
                                            const item = checklist[p.id];
                                            const matchesSearch = p.title.toLowerCase().includes(searchTerm.toLowerCase()) || p.id.toString() === searchTerm || p.code.toLowerCase().includes(searchTerm.toLowerCase());
                                            const matchesStatus = statusFilter === 'all' || item.status === statusFilter;
                                            return matchesSearch && matchesStatus;
                                        });
                                        if (filteredPoints.length === 0) return null;

                                        return (
                                            <div key={zone.id} className="bg-white rounded-xl shadow-sm border border-slate-200 overflow-hidden">
                                                <div className="bg-slate-900 px-5 py-3 text-white flex items-center justify-between">
                                                    <h3 className="font-bold text-sm sm:text-base flex items-center gap-2">
                                                        <i className={`${zone.icon} text-xl text-amber-400`}></i>
                                                        {zone.name}
                                                    </h3>
                                                    <span className="text-xs bg-slate-800 text-amber-400 border border-slate-700 px-2.5 py-1 rounded-full font-mono font-bold">
                                                        {filteredPoints.length} รายการ
                                                    </span>
                                                </div>
                                                <div className="divide-y divide-slate-100">
                                                    {filteredPoints.map(point => {
                                                        const itemData = checklist[point.id];
                                                        const isUnlocked = itemData.isScanned;
                                                        return (
                                                            <div key={point.id} className={`p-4 sm:p-5 transition-colors relative ${!isUnlocked ? 'bg-slate-50/80 border-l-4 border-l-slate-400' : itemData.status === 'issue' ? 'bg-rose-50/40 border-l-4 border-l-rose-500' : 'bg-white border-l-4 border-l-emerald-500'}`}>
                                                                <div className="flex flex-col lg:flex-row lg:items-start justify-between gap-4">
                                                                    <div className="flex-1 space-y-3">
                                                                        <div className="flex items-start justify-between gap-2">
                                                                            <div className="flex items-start gap-2">
                                                                                <span className="bg-slate-200 text-slate-800 text-xs font-bold px-2 py-0.5 rounded mt-0.5 flex-shrink-0 font-mono">#{point.id}</span>
                                                                                <div>
                                                                                    <h4 className="font-bold text-slate-900 text-base leading-snug">{point.title}</h4>
                                                                                    <div className="flex items-center gap-2 text-xs text-slate-500 mt-0.5 font-mono">
                                                                                        <span className="bg-slate-100 text-slate-600 px-1.5 py-0.5 rounded border border-slate-200">ID: {point.code}</span>
                                                                                    </div>
                                                                                </div>
                                                                            </div>
                                                                            {isUnlocked ? (
                                                                                <div className="flex items-center gap-1 bg-emerald-50 text-emerald-800 border border-emerald-200 text-xs font-mono px-2.5 py-1 rounded-full flex-shrink-0 font-bold">
                                                                                    <i className="fa-solid fa-clock text-emerald-600"></i>
                                                                                    <span>สแกนเมื่อ: {itemData.scanTimestamp} น.</span>
                                                                                </div>
                                                                            ) : (
                                                                                <div className="flex items-center gap-1 bg-amber-100 text-amber-900 border border-amber-300 text-xs font-medium px-2 py-1 rounded-full flex-shrink-0">
                                                                                    <i className="fa-solid fa-lock text-sm text-amber-700"></i>
                                                                                    <span>ต้องสแกน QR Code</span>
                                                                                </div>
                                                                            )}
                                                                        </div>
                                                                        {!isUnlocked ? (
                                                                            <div className="bg-amber-50/70 p-3 rounded-lg border border-amber-200 flex flex-col sm:flex-row sm:items-center justify-between gap-3">
                                                                                <span className="text-xs text-amber-900 font-medium flex items-center gap-1.5">
                                                                                    <i className="fa-solid fa-circle-info text-amber-600"></i>
                                                                                    <span>ยังไม่ได้ตรวจสอบ: กรุณาสแกน QR Code หรืออัปโหลดรูปภาพ ณ จุดตรวจจริง</span>
                                                                                </span>
                                                                                <button onClick={() => { setCameraError(null); setActiveScanPoint(point); setIsGlobalScanMode(false); }} className="px-3.5 py-1.5 bg-slate-900 hover:bg-slate-800 text-amber-400 font-bold text-xs rounded-lg shadow flex items-center gap-1.5">
                                                                                    <i className="fa-solid fa-qrcode"></i>
                                                                                    <span>เปิดกล้องสแกนจุดนี้</span>
                                                                                </button>
                                                                            </div>
                                                                        ) : (
                                                                            <div className="space-y-3 pt-1">
                                                                                <div className="flex flex-wrap items-center gap-2">
                                                                                    <button onClick={() => setChecklist(prev => ({ ...prev, [point.id]: { ...prev[point.id], status: 'normal' } }))} className={`px-3.5 py-1.5 rounded-lg text-xs font-bold flex items-center gap-1.5 transition-all ${itemData.status === 'normal' ? 'bg-emerald-600 text-white shadow-sm' : 'bg-slate-100 text-slate-600 hover:bg-slate-200'}`}>
                                                                                        <i className="fa-solid fa-circle-check"></i><span>ปกติ</span>
                                                                                    </button>
                                                                                    <button onClick={() => setChecklist(prev => ({ ...prev, [point.id]: { ...prev[point.id], status: 'issue' } }))} className={`px-3.5 py-1.5 rounded-lg text-xs font-bold flex items-center gap-1.5 transition-all ${itemData.status === 'issue' ? 'bg-rose-600 text-white shadow-sm' : 'bg-slate-100 text-slate-600 hover:bg-slate-200'}`}>
                                                                                        <i className="fa-solid fa-triangle-exclamation"></i><span>พบจุดแก้ไข</span>
                                                                                    </button>
                                                                                </div>
                                                                                {itemData.status === 'issue' && (
                                                                                    <div className="p-3 bg-rose-50 rounded-lg border border-rose-200 space-y-2">
                                                                                        <div className="flex items-center gap-2">
                                                                                            <span className="text-xs font-bold text-rose-900">ระดับความรุนแรง:</span>
                                                                                            {['Low', 'Medium', 'High', 'Critical'].map(level => (
                                                                                                <button key={level} onClick={() => setChecklist(prev => ({ ...prev, [point.id]: { ...prev[point.id], severity: level } }))} className={`px-2 py-0.5 rounded text-xs font-bold ${itemData.severity === level ? 'bg-rose-600 text-white' : 'bg-white text-rose-700 border border-rose-300'}`}>{level}</button>
                                                                                            ))}
                                                                                        </div>
                                                                                        <input type="text" placeholder="ระบุข้อบกพร่องที่พบ หรือ สิ่งที่ต้องดำเนินการแก้ไข..." value={itemData.note} onChange={(e) => setChecklist(prev => ({ ...prev, [point.id]: { ...prev[point.id], note: e.target.value } }))} className="w-full text-xs p-2 bg-white border border-rose-300 rounded focus:outline-none focus:ring-1 focus:ring-rose-500" />
                                                                                    </div>
                                                                                )}
                                                                            </div>
                                                                        )}
                                                                    </div>
                                                                </div>
                                                            </div>
                                                        );
                                                    })}
                                                </div>
                                            </div>
                                        );
                                    })}
                                </div>
                            </div>
                        )}

                        {activeTab === 'report' && (
                            <div className="print-area space-y-6">
                                <div className="no-print bg-slate-900 text-white p-4 rounded-xl flex flex-col sm:flex-row items-center justify-between gap-3">
                                    <div className="flex items-center space-x-2">
                                        <i className="fa-solid fa-print text-xl text-amber-400"></i>
                                        <span className="text-sm font-bold">รายงานสรุปทางการ พร้อมพิมพ์ PDF และส่งออก Excel</span>
                                    </div>
                                    <div className="flex items-center space-x-2">
                                        <button onClick={handleExportCSV} className="px-4 py-2 bg-slate-800 hover:bg-slate-700 text-slate-200 text-xs font-bold rounded-lg border border-slate-700 flex items-center gap-1.5">
                                            <i className="fa-solid fa-download text-emerald-400"></i><span>ส่งออก CSV (Excel)</span>
                                        </button>
                                        <button onClick={handlePrintReport} className="px-4 py-2 bg-amber-500 hover:bg-amber-400 text-slate-950 text-xs font-bold rounded-lg flex items-center gap-1.5 shadow">
                                            <i className="fa-solid fa-print"></i><span>พิมพ์รายงาน / บันทึก PDF</span>
                                        </button>
                                    </div>
                                </div>
                                <div className="bg-white p-6 sm:p-8 rounded-xl shadow-md border border-slate-200 space-y-6">
                                    <div className="border-b border-slate-300 pb-4">
                                        <div className="flex justify-between items-start">
                                            <div>
                                                <h2 className="text-xl font-bold text-slate-900">รายงานผลการเดินตรวจความปลอดภัยประจำวัน (Safety Inspection Checklist)</h2>
                                                <p className="text-xs text-slate-600 mt-1 font-semibold">ตรวจสอบครอบคลุมทั้ง 46 จุดตรวจ (1 FLOOR, 2 FLOOR, OUTDOOR)</p>
                                            </div>
                                            <div className="text-right text-xs text-slate-600 font-mono">
                                                <p className="font-bold text-slate-900">วันที่: {inspectorInfo.date}</p>
                                                <p className="mt-1 font-bold text-amber-800">{inspectorInfo.shift}</p>
                                            </div>
                                        </div>
                                        <div className="grid grid-cols-2 sm:grid-cols-4 gap-3 mt-4 text-xs bg-slate-50 p-3 rounded-lg border border-slate-200">
                                            <div><span className="font-bold">ผู้ตรวจความปลอดภัย:</span> {inspectorInfo.name}</div>
                                            <div><span className="font-bold">สังกัด:</span> {inspectorInfo.department}</div>
                                            <div><span className="font-bold">สแกน QR แล้ว:</span> {stats.scannedCount} / 46 จุด</div>
                                            <div><span className="font-bold">พบจุดต้องแก้ไข:</span> <span className="text-rose-600 font-bold">{stats.issue} จุด</span></div>
                                        </div>
                                    </div>
                                    <div className="space-y-6">
                                        {CHECKPOINT_ZONES.map(zone => (
                                            <div key={zone.id} className="space-y-2">
                                                <h3 className="font-bold text-xs sm:text-sm text-slate-900 bg-slate-100 p-2 rounded border border-slate-200">{zone.name}</h3>
                                                <table className="w-full text-xs text-left border-collapse border border-slate-300">
                                                    <thead>
                                                        <tr className="bg-slate-800 text-white">
                                                            <th className="p-2 border border-slate-400 w-12 text-center">จุดที่</th>
                                                            <th className="p-2 border border-slate-400 w-28">รหัส</th>
                                                            <th className="p-2 border border-slate-400">ชื่อจุดตรวจ</th>
                                                            <th className="p-2 border border-slate-400 w-24 text-center">เวลาสแกน</th>
                                                            <th className="p-2 border border-slate-400 w-24 text-center">ผลการตรวจ</th>
                                                            <th className="p-2 border border-slate-400">หมายเหตุ / สิ่งที่ต้องแก้ไข</th>
                                                        </tr>
                                                    </thead>
                                                    <tbody>
                                                        {zone.points.map(point => {
                                                            const item = checklist[point.id];
                                                            return (
                                                                <tr key={point.id} className={item.status === 'issue' ? 'bg-rose-50 font-semibold' : ''}>
                                                                    <td className="p-2 border border-slate-300 text-center font-bold">{point.id}</td>
                                                                    <td className="p-2 border border-slate-300 font-mono">{point.code}</td>
                                                                    <td className="p-2 border border-slate-300">{point.title}</td>
                                                                    <td className="p-2 border border-slate-300 text-center font-mono">{item.scanTimestamp || '-'}</td>
                                                                    <td className="p-2 border border-slate-300 text-center">
                                                                        {item.status === 'normal' ? <span className="text-emerald-700 font-bold">ปกติ</span> : item.status === 'issue' ? <span className="text-rose-700 font-bold">ต้องแก้ไข ({item.severity})</span> : <span className="text-slate-400">รอตรวจ</span>}
                                                                    </td>
                                                                    <td className="p-2 border border-slate-300">{item.note || '-'}</td>
                                                                </tr>
                                                            );
                                                        })}
                                                    </tbody>
                                                </table>
                                            </div>
                                        ))}
                                    </div>
                                    <div className="pt-10 mt-8 grid grid-cols-2 gap-8 text-center text-xs text-slate-700 border-t-2 border-slate-300">
                                        <div className="space-y-12">
                                            <div>ลงชื่อ....................................................... ผู้รายงาน</div>
                                            <div className="font-bold">( {inspectorInfo.name} )</div>
                                            <div>เจ้าหน้าที่ตรวจความปลอดภัยประจำรอบ</div>
                                        </div>
                                        <div className="space-y-12">
                                            <div>ลงชื่อ....................................................... OM</div>
                                            <div className="font-bold">( ....................................................... )</div>
                                            <div>ผู้ตรวจสอบ / ผู้จัดการฝ่ายปฏิบัติการ (OM)</div>
                                        </div>
                                    </div>
                                </div>
                            </div>
                        )}

                        {activeTab === 'qrgenerator' && (
                            <div className="space-y-4 no-print">
                                <div className="bg-slate-900 text-white p-4 rounded-xl flex items-center justify-between">
                                    <div>
                                        <h3 className="font-bold text-base">ป้าย QR Code สำหรับติดประจำจุดตรวจทั้ง 46 จุด</h3>
                                        <p className="text-xs text-slate-400">สั่งพิมพ์ป้าย QR Code เพื่อให้นำไปแปะไว้ที่ตู้ หรือพื้นที่ตรวจจริง</p>
                                    </div>
                                    <button onClick={handlePrintReport} className="px-4 py-2 bg-amber-500 hover:bg-amber-400 text-slate-950 font-bold text-xs rounded-lg flex items-center gap-1.5">
                                        <i className="fa-solid fa-print"></i><span>พิมพ์ป้าย QR ทั้งหมด</span>
                                    </button>
                                </div>
                                <div className="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-4">
                                    {CHECKPOINT_ZONES.flatMap(z => z.points).map(point => (
                                        <div key={point.id} className="bg-white p-4 rounded-xl border border-slate-300 flex flex-col items-center text-center space-y-3">
                                            <span className="text-xs font-bold text-slate-400 font-mono">จุดตรวจที่ #{point.id}</span>
                                            <h4 className="font-bold text-slate-900 text-sm">{point.title}</h4>
                                            <div className="p-3 bg-slate-50 rounded-lg border border-slate-200">
                                                <QRCodeLabel point={point} />
                                            </div>
                                            <div className="text-xs font-mono text-slate-600 font-bold bg-slate-100 px-2 py-0.5 rounded">{point.code}</div>
                                        </div>
                                    ))}
                                </div>
                            </div>
                        )}
                    </main>

                    {(activeScanPoint || isGlobalScanMode) && (
                        <div className="fixed inset-0 z-50 bg-slate-950/85 backdrop-blur-xs flex items-center justify-center p-4 no-print overflow-y-auto">
                            <div className="bg-slate-900 text-white rounded-2xl max-w-md w-full p-5 space-y-4 text-center relative border border-slate-800 shadow-2xl my-auto">
                                <button onClick={() => { stopCamera(); setActiveScanPoint(null); setIsGlobalScanMode(false); }} className="absolute top-4 right-4 text-slate-400 hover:text-white bg-slate-800 p-1.5 rounded-full">
                                    <i className="fa-solid fa-xmark"></i>
                                </button>
                                <div className="pr-8 pl-8">
                                    <span className="text-xs font-bold text-amber-400 uppercase tracking-wider flex items-center justify-center gap-1">
                                        <i className="fa-solid fa-expand"></i><span>สแกน QR ณ จุดตรวจจริง</span>
                                    </span>
                                    <h3 className="text-base font-bold text-white mt-2">
                                        {activeScanPoint ? `จุดที่ ${activeScanPoint.id}: ${activeScanPoint.title}` : 'สแกนป้าย QR โซนใดก็ได้'}
                                    </h3>
                                    {activeScanPoint && <p className="text-xs text-slate-400 mt-1">รหัสที่ต้องอ่าน: {activeScanPoint.code}</p>}
                                </div>

                                {stats.scannedCount < 46 ? (
                                    <>
                                        <div className="relative aspect-square w-full bg-black rounded-xl overflow-hidden border-2 border-amber-500/80 flex items-center justify-center">
                                            <video ref={videoRef} autoPlay playsInline muted webkit-playsinline="true" className={`w-full h-full object-cover ${cameraActive ? 'block' : 'hidden'}`} />
                                            <canvas ref={canvasRef} className="hidden" />
                                            {cameraActive && (
                                                <>
                                                    <div className="absolute inset-x-6 top-1/2 h-0.5 bg-amber-400 shadow-[0_0_12px_#f59e0b] animate-pulse" />
                                                    <div className="absolute inset-8 border-2 border-dashed border-amber-400/70 rounded-lg pointer-events-none" />
                                                    <span className="absolute bottom-3 left-3 right-3 bg-black/60 rounded-lg px-2 py-1 text-xs text-white">วาง QR Code ให้อยู่ในกรอบ</span>
                                                </>
                                            )}
                                            {!cameraActive && (
                                                <div className="p-4 space-y-3 text-slate-300 text-center">
                                                    <i className="fa-solid fa-video-slash text-4xl text-amber-500 mx-auto"></i>
                                                    <p className="text-xs text-amber-300 font-medium">{cameraError || (cameraStarting ? 'กำลังเปิดกล้อง...' : 'กดปุ่มด้านล่างเพื่อเปิดกล้องหลัง')}</p>
                                                    <div className="bg-slate-800 p-2.5 rounded-lg border border-slate-700 text-[11px] text-slate-300 space-y-1">
                                                        <p className="font-bold text-amber-400">ใช้งานผ่าน LINE / มือถือ?</p>
                                                        <p>หากกล้องถูกบล็อก ให้ใช้ปุ่ม <b>"อัปโหลดรูปภาพ QR"</b> ด้านล่าง</p>
                                                    </div>
                                                </div>
                                            )}
                                        </div>
                                        <div className="space-y-2">
                                            {!cameraActive ? (
                                                <button onClick={startCamera} disabled={cameraStarting} className="w-full py-3 bg-amber-500 hover:bg-amber-400 text-slate-950 font-bold text-sm rounded-xl shadow flex items-center justify-center gap-2">
                                                    <i className="fa-solid fa-camera"></i><span>{cameraStarting ? 'กำลังเปิดกล้อง...' : 'อนุญาตและเปิดกล้องหลัง'}</span>
                                                </button>
                                            ) : (
                                                <div className="w-full py-2.5 bg-emerald-900/60 border border-emerald-700 text-emerald-200 font-bold text-xs rounded-xl flex items-center justify-center gap-2">
                                                    <i className="fa-solid fa-expand"></i><span>กำลังอ่าน QR — ขยับกล้องช้าๆ</span>
                                                </div>
                                            )}
                                            <label className="w-full py-2.5 bg-slate-800 hover:bg-slate-700 text-slate-200 font-semibold text-xs rounded-xl border border-slate-700 flex items-center justify-center gap-2 cursor-pointer transition-all">
                                                <i className="fa-solid fa-upload text-cyan-400"></i>
                                                <span>อัปโหลดรูปภาพป้าย QR Code (สำหรับ LINE Browser)</span>
                                                <input type="file" accept="image/*" onChange={handleFileUploadScan} className="hidden" />
                                            </label>
                                        </div>
                                    </>
                                ) : (
                                    <div className="py-8 text-emerald-300 font-bold">บันทึกครบ 46/46 จุดแล้ว</div>
                                )}
                            </div>
                        </div>
                    )}
                </div>
            );
        }

        const root = ReactDOM.createRoot(document.getElementById('root'));
        root.render(<App />);
    </script>
</body>
</html>
