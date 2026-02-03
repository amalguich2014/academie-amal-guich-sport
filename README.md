
<?php
session_start();
if (!isset($_SESSION['user_id'])) {
    header("Location: login.php");
    exit();
}

require_once('config/database.php');

// 1. جلب قائمة الموكلين لعرضهم في قائمة الاختيار (Dropdown)
$clients = $pdo->query("SELECT id, name FROM clients ORDER BY name ASC")->fetchAll();

$message = "";

// 2. معالجة إرسال النموذج
if ($_SERVER['REQUEST_METHOD'] == 'POST') {
    $client_id   = $_POST['client_id'];
    $case_number = $_POST['case_number'];
    $case_type   = $_POST['case_type'];
    $court_name  = $_POST['court_name'];
    $status      = $_POST['status'];

    try {
        $sql = "INSERT INTO cases (client_id, case_number, case_type, court_name, status) VALUES (?, ?, ?, ?, ?)";
        $stmt = $pdo->prepare($sql);
        $stmt->execute([$client_id, $case_number, $case_type, $court_name, $status]);
        
        $message = "<div class='alert alert-success'>تم فتح القضية بنجاح! <a href='cases.php'>عرض القضايا</a></div>";
    } catch (Exception $e) {
        $message = "<div class='alert alert-danger'>خطأ في الإدخال: " . $e->getMessage() . "</div>";
    }
}
?>

<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <title>إضافة قضية جديدة - نظام العدالة</title>
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.rtl.min.css">
    <link href="https://fonts.googleapis.com/css2?family=Tajawal:wght@400;700&display=swap" rel="stylesheet">
    <style>
        body { font-family: 'Tajawal', sans-serif; background-color: #f0f2f5; }
        .main-content { margin-right: 280px; padding: 40px; }
        .form-card { background: white; border-radius: 15px; padding: 30px; box-shadow: 0 5px 15px rgba(0,0,0,0.05); max-width: 800px; margin: auto; }
        .sidebar { width: 280px; background: #1a237e; min-height: 100vh; position: fixed; color: white; }
        .nav-link { color: white; padding: 15px 25px; display: block; text-decoration: none; }
        .nav-link:hover { background: rgba(255,255,255,0.1); }
    </style>
</head>
<body>

    <div class="sidebar">
        <div class="p-4 text-center"><h4>نظام العدالة</h4></div>
        <a href="index.php" class="nav-link">لوحة التحكم</a>
        <a href="cases.php" class="nav-link">القضايا</a>
    </div>

    <div class="main-content">
        <h2 class="fw-bold mb-4 text-center">فتح ملف قضية جديدة ⚖️</h2>
        
        <?php echo $message; ?>

        <div class="form-card">
            <form method="POST">
                <div class="row g-3">
                    <div class="col-md-12">
                        <label class="form-label fw-bold">الموكل (يجب أن يكون مسجلاً مسبقاً)</label>
                        <select name="client_id" class="form-select" required>
                            <option value="">اختر الموكل...</option>
                            <?php foreach ($clients as $client): ?>
                                <option value="<?php echo $client['id']; ?>"><?php echo htmlspecialchars($client['name']); ?></option>
                            <?php endforeach; ?>
                        </select>
                    </div>

                    <div class="col-md-6">
                        <label class="form-label fw-bold">رقم القضية / الرقم الآلي</label>
                        <input type="text" name="case_number" class="form-control" placeholder="مثلاً: 2026/102" required>
                    </div>

                    <div class="col-md-6">
                        <label class="form-label fw-bold">نوع القضية</label>
                        <select name="case_type" class="form-select">
                            <option>مدني</option>
                            <option>جنائي</option>
                            <option>أحوال شخصية</option>
                            <option>تجاري</option>
                            <option>عمالي</option>
                        </select>
                    </div>

                    <div class="col-md-6">
                        <label class="form-label fw-bold">المحكمة / الدائرة</label>
                        <input type="text" name="court_name" class="form-control" placeholder="مثلاً: محكمة الاستئناف - الدائرة 5" required>
                    </div>

                    <div class="col-md-6">
                        <label class="form-label fw-bold">الحالة الابتدائية</label>
                        <select name="status" class="form-select">
                            <option value="جارية">جارية</option>
                            <option value="متوقفة">متوقفة</option>
                        </select>
                    </div>

                    <div class="col-12 mt-4 text-center">
                        <button type="submit" class="btn btn-primary px-5 py-2 border-0 shadow" style="background: #1a237e;">
                            حفظ القضية في النظام
                        </button>
                    </div>
                </div>
            </form>
        </div>
    </div>

</body>
</html>
<?php
// 1. إدارة الجلسة والأمان
session_start();
if (!isset($_SESSION['user_id'])) {
    header("Location: login.php");
    exit();
}

// 2. الاتصال بقاعدة البيانات ومعالجة الطلب
require_once('config/database.php');

$message = "";
if ($_SERVER['REQUEST_METHOD'] == 'POST') {
    $name = $_POST['name'];
    $phone = $_POST['phone'];
    $email = $_POST['email'];
    $address = $_POST['address'];
    $id_number = $_POST['id_number'];

    try {
        $stmt = $pdo->prepare("INSERT INTO clients (name, phone, email, address, id_number) VALUES (?, ?, ?, ?, ?)");
        if ($stmt->execute([$name, $phone, $email, $address, $id_number])) {
            $message = "<div class='alert alert-success'>تمت إضافة الموكل بنجاح بنجاح</div>";
            // اختياري: التوجه لصفحة الموكلين بعد ثانية واحدة
            header("refresh:1;url=clients.php");
        }
    } catch (Exception $e) {
        $message = "<div class='alert alert-danger'>خطأ: تعذر حفظ البيانات. قد يكون رقم الهوية مسجلاً مسبقاً.</div>";
    }
}
?>

<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <title>إضافة موكل - نظام العدالة</title>
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.rtl.min.css">
    <link href="https://fonts.googleapis.com/css2?family=Tajawal:wght@300;500;800&display=swap" rel="stylesheet">
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    
    <style>
        :root {
            --law-dark: #1a237e;
            --law-gold: #c5a059;
        }

        body {
            background-color: #f0f2f5;
            font-family: 'Tajawal', sans-serif;
            min-height: 100vh;
            display: flex;
            align-items: center;
            justify-content: center;
            padding: 20px;
        }

        .form-card {
            background: white;
            max-width: 800px;
            width: 100%;
            border-radius: 25px;
            box-shadow: 0 15px 35px rgba(0,0,0,0.1);
            overflow: hidden;
            border: none;
        }

        .form-header {
            background: var(--law-dark);
            color: white;
            padding: 30px;
            text-align: center;
        }

        .form-header i {
            font-size: 40px;
            color: var(--law-gold);
            margin-bottom: 10px;
        }

        .form-body {
            padding: 40px;
        }

        .form-label {
            font-weight: 800;
            color: var(--law-dark);
            margin-bottom: 10px;
        }

        .form-control {
            border-radius: 12px;
            padding: 12px 20px;
            border: 1px solid #e0e0e0;
            transition: 0.3s;
        }

        .form-control:focus {
            border-color: var(--law-gold);
            box-shadow: 0 0 0 0.25rem rgba(197, 160, 89, 0.1);
        }

        .btn-save {
            background: var(--law-dark);
            color: white;
            border-radius: 12px;
            padding: 12px 30px;
            font-weight: bold;
            border: none;
            transition: 0.3s;
            width: 100%;
        }

        .btn-save:hover {
            background: #0d1440;
            color: var(--law-gold);
            transform: translateY(-2px);
        }

        .btn-back {
            color: var(--law-dark);
            text-decoration: none;
            font-weight: bold;
            display: inline-flex;
            align-items: center;
            margin-bottom: 20px;
        }
    </style>
</head>
<body>

    <div class="form-card">
        <div class="form-header">
            <i class="fas fa-user-plus"></i>
            <h3 class="fw-bold mb-0">تسجيل موكل جديد</h3>
            <p class="mb-0 opacity-75">أدخل البيانات الأساسية لفتح ملف الموكل</p>
        </div>

        <div class="form-body">
            <a href="clients.php" class="btn-back">
                <i class="fas fa-arrow-right ms-2"></i> العودة لقائمة الموكلين
            </a>

            <?php echo $message; ?>

            <form action="" method="POST">
                <div class="row g-4">
                    <div class="col-md-12">
                        <label class="form-label">الاسم الكامل للموكل</label>
                        <input type="text" name="name" class="form-control" placeholder="مثال: أحمد محمد علي" required>
                    </div>

                    <div class="col-md-6">
                        <label class="form-label">رقم البطاقة الوطنية / السجل</label>
                        <input type="text" name="id_number" class="form-control" placeholder="رقم التعريف الرسمي" required>
                    </div>

                    <div class="col-md-6">
                        <label class="form-label">رقم الهاتف</label>
                        <input type="tel" name="phone" class="form-control" placeholder="0600000000" required>
                    </div>

                    <div class="col-md-12">
                        <label class="form-label">البريد الإلكتروني (اختياري)</label>
                        <input type="email" name="email" class="form-control" placeholder="client@email.com">
                    </div>

                    <div class="col-md-12">
                        <label class="form-label">العنوان الشخصي / المختار</label>
                        <textarea name="address" class="form-control" rows="3" placeholder="أدخل العنوان الكامل للمراسلات"></textarea>
                    </div>

                    <div class="col-md-12 mt-5">
                        <button type="submit" class="btn-save shadow-sm">
                            <i class="fas fa-check-circle me-2"></i> حفظ بيانات الموكل
                        </button>
                    </div>
                </div>
            </form>
        </div>
    </div>

</body>
</html>
<?php
session_start();
if (!isset($_SESSION['user_id'])) {
    header("Location: login.php");
    exit();
}

require_once('config/database.php');

// 1. جلب القضايا لربط الدفعة بها
$query_cases = "SELECT cases.id, cases.case_number, clients.name 
                FROM cases 
                JOIN clients ON cases.client_id = clients.id 
                ORDER BY cases.created_at DESC";
$cases = $pdo->query($query_cases)->fetchAll();

$message = "";

// 2. معالجة الإرسال
if ($_SERVER['REQUEST_METHOD'] == 'POST') {
    $case_id = $_POST['case_id'];
    $total_fee = $_POST['total_fee']; // إجمالي الاتفاق
    $amount_paid = $_POST['amount_paid']; // الدفعة الحالية
    $payment_date = $_POST['payment_date'];
    $notes = $_POST['notes'];

    try {
        $pdo->beginTransaction();

        // تحديث أو إدخال إجمالي الأتعاب في جدول case_finances
        $stmt_fee = $pdo->prepare("INSERT INTO case_finances (case_id, total_fee) 
                                   VALUES (?, ?) 
                                   ON DUPLICATE KEY UPDATE total_fee = ?");
        $stmt_fee->execute([$case_id, $total_fee, $total_fee]);

        // تسجيل الدفعة في جدول payments
        if ($amount_paid > 0) {
            $stmt_pay = $pdo->prepare("INSERT INTO payments (case_id, amount_paid, payment_date, notes) VALUES (?, ?, ?, ?)");
            $stmt_pay->execute([$case_id, $amount_paid, $payment_date, $notes]);
        }

        $pdo->commit();
        $message = "<div class='alert alert-success shadow-sm'>تم تسجيل البيانات المالية بنجاح! <a href='finance.php' class='fw-bold'>عرض السجلات</a></div>";
    } catch (Exception $e) {
        $pdo->rollBack();
        $message = "<div class='alert alert-danger'>خطأ: " . $e->getMessage() . "</div>";
    }
}
?>

<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <title>تسجيل دفعة مالية - نظام العدالة</title>
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.rtl.min.css">
    <link href="https://fonts.googleapis.com/css2?family=Tajawal:wght@400;700&display=swap" rel="stylesheet">
    <style>
        body { font-family: 'Tajawal', sans-serif; background-color: #f0f2f5; }
        .main-content { margin-right: 280px; padding: 40px; }
        .finance-card { background: white; border-radius: 20px; padding: 35px; box-shadow: 0 10px 25px rgba(0,0,0,0.05); max-width: 800px; margin: auto; border-top: 5px solid #28a745; }
        .sidebar { width: 280px; background: #1a237e; min-height: 100vh; position: fixed; color: white; }
        .nav-link { color: white; padding: 15px 25px; display: block; text-decoration: none; }
        .nav-link:hover { background: rgba(255,255,255,0.1); }
        .form-label { font-weight: bold; color: #2c3e50; }
    </style>
</head>
<body>

    <div class="sidebar">
        <div class="p-4 text-center"><h4>نظام العدالة</h4></div>
        <a href="index.php" class="nav-link">لوحة التحكم</a>
        <a href="finance.php" class="nav-link">الحسابات المالية</a>
    </div>

    <div class="main-content">
        <div class="text-center mb-5">
            <h2 class="fw-bold">تسجيل معاملة مالية 💰</h2>
            <p class="text-muted">إدارة أتعاب القضايا وتحصيل الدفعات</p>
        </div>

        <div class="finance-card">
            <?php echo $message; ?>
            <form method="POST">
                <div class="row g-4">
                    <div class="col-12">
                        <label class="form-label">اختر القضية</label>
                        <select name="case_id" class="form-select form-select-lg" required>
                            <option value="">-- اختر القضية والموكل --</option>
                            <?php foreach ($cases as $case): ?>
                                <option value="<?php echo $case['id']; ?>">
                                    قضية: <?php echo $case['case_number']; ?> - (<?php echo $case['name']; ?>)
                                </option>
                            <?php endforeach; ?>
                        </select>
                    </div>

                    <div class="col-md-6">
                        <label class="form-label">إجمالي الأتعاب المتفق عليها</label>
                        <input type="number" name="total_fee" class="form-control" placeholder="0.00" required step="0.01">
                        <small class="text-muted">سيتم تحديث الإجمالي إذا كانت القضية مسجلة مسبقاً.</small>
                    </div>

                    <div class="col-md-6">
                        <label class="form-label">المبلغ المدفوع الآن</label>
                        <input type="number" name="amount_paid" class="form-control" placeholder="0.00" step="0.01">
                    </div>

                    <div class="col-md-6">
                        <label class="form-label">تاريخ الدفعة</label>
                        <input type="date" name="payment_date" class="form-control" value="<?php echo date('Y-m-d'); ?>">
                    </div>

                    <div class="col-md-6">
                        <label class="form-label">ملاحظات الصرف</label>
                        <input type="text" name="notes" class="form-control" placeholder="مثال: دفعة أولى، كاش، شيك رقم...">
                    </div>

                    <div class="col-12 text-center mt-4">
                        <button type="submit" class="btn btn-success btn-lg px-5 shadow">
                            <i class="fas fa-save me-2"></i> حفظ البيانات المالية
                        </button>
                    </div>
                </div>
            </form>
        </div>
    </div>

</body>
</html>
<?php
session_start();
if (!isset($_SESSION['user_id'])) {
    header("Location: login.php");
    exit();
}

require_once('config/database.php');

// 1. جلب معرف القضية من الرابط (Query String)
$case_id = isset($_GET['case_id']) ? $_GET['case_id'] : null;

if (!$case_id) {
    die("خطأ: يجب تحديد القضية لإضافة جلسة لها.");
}

// 2. معالجة إرسال النموذج (حفظ الجلسة في قاعدة البيانات)
$message = "";
if ($_SERVER['REQUEST_METHOD'] == 'POST') {
    $session_date = $_POST['session_date'];
    $session_time = $_POST['session_time'];
    $requirements = $_POST['requirements'];
    $court_name   = $_POST['court_name'];

    try {
        // الاستعلام لإدخال البيانات
        $stmt = $pdo->prepare("INSERT INTO sessions (case_id, session_date, session_time, requirements, court_name) VALUES (?, ?, ?, ?, ?)");
        if ($stmt->execute([$case_id, $session_date, $session_time, $requirements, $court_name])) {
            $message = "<div class='alert alert-success shadow-sm'>
                            <i class='fas fa-check-circle me-2'></i> تم إضافة الجلسة بنجاح! 
                            <a href='view_case.php?id=$case_id' class='fw-bold text-decoration-none'>العودة لملف القضية</a>
                        </div>";
        }
    } catch (Exception $e) {
        $message = "<div class='alert alert-danger shadow-sm'>
                        <i class='fas fa-exclamation-triangle me-2'></i> خطأ في الإضافة: " . $e->getMessage() . "
                    </div>";
    }
}

// 3. جلب بيانات القضية لعرض الرقم في العناوين
$stmt_case = $pdo->prepare("SELECT case_number FROM cases WHERE id = ?");
$stmt_case->execute([$case_id]);
$case_data = $stmt_case->fetch();
?>

<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>إضافة جلسة جديدة | نظام العدالة</title>
    
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.rtl.min.css">
    <link href="https://fonts.googleapis.com/css2?family=Tajawal:wght@400;700&display=swap" rel="stylesheet">
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    
    <style>
        body { 
            font-family: 'Tajawal', sans-serif; 
            background-color: #f4f7f6; 
            padding-top: 50px;
        }
        .form-card { 
            border: none; 
            border-radius: 20px; 
            box-shadow: 0 15px 35px rgba(0,0,0,0.1); 
            overflow: hidden;
        }
        .card-header-law {
            background-color: #1a237e;
            color: white;
            padding: 25px;
            text-align: center;
        }
        .btn-law { 
            background-color: #1a237e; 
            color: white; 
            font-weight: bold;
            padding: 12px;
            transition: 0.3s;
        }
        .btn-law:hover { 
            background-color: #c5a059; 
            color: white; 
            transform: translateY(-2px);
        }
        .form-label { color: #1a237e; }
    </style>
</head>
<body>

<div class="container mb-5">
    <div class="row justify-content-center">
        <div class="col-md-6">
            <div class="card form-card">
                <div class="card-header-law">
                    <i class="fas fa-calendar-plus fa-3x mb-3" style="color: #c5a059;"></i>
                    <h3 class="fw-bold m-0">إضافة جلسة جديدة</h3>
                    <p class="opacity-75 m-0 mt-2 small">قضية رقم: <?php echo htmlspecialchars($case_data['case_number']); ?></p>
                </div>
                
                <div class="card-body p-4">
                    <?php echo $message; ?>

                    <form method="POST">
                        <div class="row">
                            <div class="col-md-6 mb-3">
                                <label class="form-label fw-bold"><i class="far fa-calendar-alt me-1"></i> تاريخ الجلسة</label>
                                <input type="date" name="session_date" class="form-control" required>
                            </div>
                            <div class="col-md-6 mb-3">
                                <label class="form-label fw-bold"><i class="far fa-clock me-1"></i> الساعة</label>
                                <input type="time" name="session_time" class="form-control" required>
                            </div>
                        </div>

                        <div class="mb-3">
                            <label class="form-label fw-bold"><i class="fas fa-map-marker-alt me-1"></i> المحكمة / الدائرة</label>
                            <input type="text" name="court_name" class="form-control" placeholder="مثلاً: محكمة الأسرة - الدائرة 2">
                        </div>

                        <div class="mb-4">
                            <label class="form-label fw-bold"><i class="fas fa-edit me-1"></i> المطلوب / الإجراء المتوقع</label>
                            <textarea name="requirements" class="form-control" rows="4" placeholder="اكتب تفاصيل ما يجب فعله في هذه الجلسة..."></textarea>
                        </div>

                        <div class="d-grid gap-2">
                            <button type="submit" class="btn btn-law btn-lg shadow-sm">
                                <i class="fas fa-save me-2"></i> حفظ بيانات الجلسة
                            </button>
                            <a href="view_case.php?id=<?php echo $case_id; ?>" class="btn btn-light text-muted border-0 mt-2">
                                إلغاء والعودة
                            </a>
                        </div>
                    </form>
                </div>
            </div>
            
            <p class="text-center mt-4 text-muted small">نظام العدالة الرقمي - جميع الحقوق محفوظة</p>
        </div>
    </div>
</div>

<script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/js/bootstrap.bundle.min.js"></script>
</body>
</html>
<?php
session_start();
if (!isset($_SESSION['user_id'])) {
    header("Location: login.php");
    exit();
}

require_once('config/database.php');

$message = "";

if ($_SERVER['REQUEST_METHOD'] == 'POST') {
    $task_title = $_POST['task_title'];
    $due_date = $_POST['due_date'];
    $priority = $_POST['priority'];

    try {
        $sql = "INSERT INTO tasks (task_title, due_date, priority) VALUES (?, ?, ?)";
        $stmt = $pdo->prepare($sql);
        $stmt->execute([$task_title, $due_date, $priority]);
        $message = "<div class='alert alert-success'>تم إضافة المهمة بنجاح!</div>";
    } catch (Exception $e) {
        $message = "<div class='alert alert-danger'>خطأ في الإضافة: " . $e->getMessage() . "</div>";
    }
}
?>

<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <title>إضافة مهمة جديدة</title>
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.rtl.min.css">
    <link href="https://fonts.googleapis.com/css2?family=Tajawal:wght@400;700&display=swap" rel="stylesheet">
    <style>
        body { font-family: 'Tajawal', sans-serif; background-color: #f4f7f6; padding: 40px; }
        .form-card { background: white; border-radius: 15px; box-shadow: 0 10px 30px rgba(0,0,0,0.05); max-width: 600px; margin: auto; padding: 30px; }
        .btn-save { background-color: #1a237e; color: white; }
    </style>
</head>
<body>

<div class="form-card">
    <h4 class="fw-bold mb-4 text-center">إضافة إجراء أو مهمة عاجلة</h4>
    <?php echo $message; ?>
    
    <form method="POST">
        <div class="mb-3">
            <label class="form-label">عنوان المهمة / الإجراء</label>
            <input type="text" name="task_title" class="form-control" placeholder="مثلاً: تقديم مذكرة رد على الخبير" required>
        </div>
        
        <div class="row">
            <div class="col-md-6 mb-3">
                <label class="form-label">تاريخ الاستحقاق</label>
                <input type="date" name="due_date" class="form-control" required>
            </div>
            <div class="col-md-6 mb-3">
                <label class="form-label">درجة الأهمية</label>
                <select name="priority" class="form-select">
                    <option value="عادي">عادي</option>
                    <option value="عاجل">عاجل 🔥</option>
                    <option value="منخفض">منخفض</option>
                </select>
            </div>
        </div>

        <div class="mt-4 d-flex gap-2">
            <button type="submit" class="btn btn-save w-100 py-2">حفظ المهمة</button>
            <a href="index.php" class="btn btn-outline-secondary w-100 py-2">إلغاء</a>
        </div>
    </form>
</div>

</body>
</html>
<?php
session_start();
if (!isset($_SESSION['user_id'])) { header("Location: login.php"); exit(); }
require_once('config/database.php');

// جلب الملفات المرفوعة مع بيانات القضية
$query = "SELECT archive.*, cases.case_number, clients.name as client_name 
          FROM archive 
          JOIN cases ON archive.case_id = cases.id 
          JOIN clients ON cases.client_id = clients.id 
          ORDER BY archive.upload_date DESC";
$files = $pdo->query($query)->fetchAll();
?>

<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <title>الأرشيف الرقمي - نظام العدالة</title>
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.rtl.min.css">
    <link href="https://fonts.googleapis.com/css2?family=Tajawal:wght@400;700&display=swap" rel="stylesheet">
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <style>
        body { font-family: 'Tajawal', sans-serif; background-color: #f8f9fa; }
        .main-content { margin-right: 280px; padding: 40px; }
        .sidebar { width: 280px; background: #1a237e; min-height: 100vh; position: fixed; color: white; }
        .nav-link { color: white; padding: 15px 25px; display: block; text-decoration: none; }
        .nav-link.active { background: #c5a059; }
        .file-icon { font-size: 2rem; color: #1a237e; }
    </style>
</head>
<body>

    <div class="sidebar">
        <div class="p-4 text-center"><h4>نظام العدالة</h4></div>
        <a href="index.php" class="nav-link">لوحة التحكم</a>
        <a href="archive.php" class="nav-link active">الأرشيف الرقمي</a>
    </div>

    <div class="main-content">
        <div class="d-flex justify-content-between align-items-center mb-4">
            <h2 class="fw-bold">الأرشيف الرقمي والمستندات 📄</h2>
            <a href="upload_file.php" class="btn btn-primary px-4 shadow-sm" style="background: #1a237e; border: none;">
                <i class="fas fa-upload me-2"></i> رفع مستند جديد
            </a>
        </div>

        <div class="row g-4">
            <?php foreach ($files as $file): ?>
            <div class="col-md-4">
                <div class="card h-100 shadow-sm border-0 rounded-4 p-3">
                    <div class="text-center mb-3">
                        <i class="fas fa-file-pdf file-icon"></i>
                    </div>
                    <div class="card-body p-0 text-center">
                        <h6 class="fw-bold mb-1"><?php echo htmlspecialchars($file['file_name']); ?></h6>
                        <p class="small text-muted mb-2">قضية: <?php echo $file['case_number']; ?> - <?php echo $file['client_name']; ?></p>
                        <p class="small text-secondary mb-3"><?php echo $file['file_description']; ?></p>
                        <div class="d-grid gap-2">
                            <a href="<?php echo $file['file_path']; ?>" target="_blank" class="btn btn-sm btn-outline-primary">
                                <i class="fas fa-eye me-1"></i> عرض المستند
                            </a>
                        </div>
                    </div>
                </div>
            </div>
            <?php endforeach; ?>
            <?php if(empty($files)) echo "<p class='text-center py-5'>لا توجد مستندات مؤرشفة حالياً</p>"; ?>
        </div>
    </div>
</body>
</html>
<?php
session_start();
require_once('config/database.php');

if (!isset($_GET['case_id'])) { die("قضية غير محددة"); }
$case_id = $_GET['case_id'];

// معالجة رفع الملف
if (isset($_FILES['document'])) {
    $file = $_FILES['document'];
    $doc_name = $_POST['doc_name'];
    $ext = pathinfo($file['name'], PATHINFO_EXTENSION);
    $new_name = "doc_" . time() . "." . $ext;
    $target = "uploads/cases/" . $new_name;

    if (move_uploaded_file($file['tmp_name'], $target)) {
        $stmt = $pdo->prepare("INSERT INTO documents (case_id, doc_title, file_path) VALUES (?, ?, ?)");
        $stmt->execute([$case_id, $doc_name, $target]);
        $success = "تم رفع المستند بنجاح";
    }
}

// جلب المستندات الحالية
$docs = $pdo->prepare("SELECT * FROM documents WHERE case_id = ?");
$docs->execute([$case_id]);
$all_docs = $docs->fetchAll();
?>

<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.rtl.min.css">
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <style>
        .doc-card { transition: 0.3s; border-radius: 10px; border: 1px solid #ddd; }
        .doc-card:hover { transform: scale(1.02); box-shadow: 0 5px 15px rgba(0,0,0,0.1); }
        .file-icon { font-size: 3rem; color: #c5a059; }
    </style>
</head>
<body class="bg-light p-4">

<div class="container">
    <div class="card p-4 shadow-sm mb-4">
        <h4 class="fw-bold mb-4"><i class="fas fa-folder-open me-2 text-primary"></i> إدارة مستندات القضية</h4>
        
        <form method="POST" enctype="multipart/form-data" class="row g-3">
            <div class="col-md-5">
                <input type="text" name="doc_name" class="form-control" placeholder="اسم المستند (مثلاً: توكيل رسمي)" required>
            </div>
            <div class="col-md-5">
                <input type="file" name="document" class="form-control" required>
            </div>
            <div class="col-md-2">
                <button type="submit" class="btn btn-primary w-100">رفع الملف</button>
            </div>
        </form>
    </div>

    <div class="row g-3">
        <?php foreach($all_docs as $d): ?>
        <div class="col-md-3">
            <div class="doc-card p-3 bg-white text-center">
                <i class="fas fa-file-pdf file-icon mb-2"></i>
                <h6 class="fw-bold text-truncate"><?php echo htmlspecialchars($d['doc_title']); ?></h6>
                <div class="mt-3">
                    <a href="<?php echo $d['file_path']; ?>" class="btn btn-sm btn-outline-dark" target="_blank">عرض</a>
                    <a href="delete_doc.php?id=<?php echo $d['id']; ?>" class="btn btn-sm btn-outline-danger" onclick="return confirm('هل أنت متأكد؟')">حذف</a>
                </div>
            </div>
        </div>
        <?php endforeach; ?>
    </div>
</div>

</body>
</html>
<?php
session_start();
if (!isset($_SESSION['user_id'])) {
    header("Location: login.php");
    exit();
}

require_once('config/database.php');

// جلب القضايا مع أسماء الموكلين
$query = "SELECT cases.*, clients.name as client_name 
          FROM cases 
          JOIN clients ON cases.client_id = clients.id 
          ORDER BY cases.created_at DESC";
$cases = $pdo->query($query)->fetchAll();
?>

<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <title>إدارة القضايا - نظام العدالة</title>
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.rtl.min.css">
    <link href="https://fonts.googleapis.com/css2?family=Tajawal:wght@400;700&display=swap" rel="stylesheet">
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <style>
        body { font-family: 'Tajawal', sans-serif; background-color: #f0f2f5; }
        .main-content { margin-right: 280px; padding: 40px; }
        .case-card { background: white; border-radius: 15px; box-shadow: 0 5px 15px rgba(0,0,0,0.05); }
        .status-badge-جارية { background-color: #e3f2fd; color: #0d47a1; }
        .status-badge-منتهية { background-color: #e8f5e9; color: #1b5e20; }
        .status-badge-متوقفة { background-color: #fff3e0; color: #e65100; }
        /* استنساخ تنسيق السايدبار من index.php لضمان التناسق */
        .sidebar { width: 280px; background: #1a237e; min-height: 100vh; color: white; position: fixed; }
        .nav-link { color: rgba(255,255,255,0.8); padding: 15px 25px; text-decoration: none; display: block; }
        .nav-link:hover, .nav-link.active { background: rgba(255,255,255,0.05); color: #c5a059; border-right: 4px solid #c5a059; }
    </style>
</head>
<body>

    <div class="sidebar">
        <div class="p-4 text-center">
            <i class="fas fa-balance-scale fa-3x mb-2" style="color: #c5a059;"></i>
            <h4 class="fw-bold">نظام العدالة</h4>
        </div>
        <div class="mt-4">
            <a href="index.php" class="nav-link"><i class="fas fa-th-large me-2"></i> لوحة التحكم</a>
            <a href="clients.php" class="nav-link"><i class="fas fa-users me-2"></i> الموكلين</a>
            <a href="cases.php" class="nav-link active"><i class="fas fa-briefcase me-2"></i> القضايا</a>
            <a href="sessions.php" class="nav-link"><i class="fas fa-gavel me-2"></i> الجلسات</a>
            <a href="logout.php" class="nav-link text-danger"><i class="fas fa-power-off me-2"></i> خروج</a>
        </div>
    </div>

    <div class="main-content">
        <div class="d-flex justify-content-between align-items-center mb-4">
            <h2 class="fw-bold">قائمة القضايا 📁</h2>
            <a href="add_case.php" class="btn btn-primary px-4 shadow-sm border-0" style="background: #1a237e;">
                <i class="fas fa-plus me-2"></i> إضافة قضية جديدة
            </a>
        </div>

        <div class="case-card p-4">
            <div class="table-responsive">
                <table class="table table-hover align-middle">
                    <thead class="table-light">
                        <tr>
                            <th>رقم القضية</th>
                            <th>الموكل</th>
                            <th>نوع القضية</th>
                            <th>المحكمة</th>
                            <th>الحالة</th>
                            <th>الإجراءات</th>
                        </tr>
                    </thead>
                    <tbody>
                        <?php foreach ($cases as $case): ?>
                        <tr>
                            <td class="fw-bold">#<?php echo htmlspecialchars($case['case_number']); ?></td>
                            <td><?php echo htmlspecialchars($case['client_name']); ?></td>
                            <td><?php echo htmlspecialchars($case['case_type']); ?></td>
                            <td><?php echo htmlspecialchars($case['court_name']); ?></td>
                            <td>
                                <span class="badge rounded-pill px-3 py-2 status-badge-<?php echo $case['status']; ?>">
                                    <?php echo $case['status']; ?>
                                </span>
                            </td>
                            <td>
                                <div class="btn-group">
                                    <a href="view_case.php?id=<?php echo $case['id']; ?>" class="btn btn-sm btn-outline-primary"><i class="fas fa-eye"></i></a>
                                    <a href="edit_case.php?id=<?php echo $case['id']; ?>" class="btn btn-sm btn-outline-secondary"><i class="fas fa-edit"></i></a>
                                </div>
                            </td>
                        </tr>
                        <?php endforeach; ?>
                        <?php if(empty($cases)) echo "<tr><td colspan='6' class='text-center py-4 text-muted'>لا توجد قضايا مسجلة حالياً</td></tr>"; ?>
                    </tbody>
                </table>
            </div>
        </div>
    </div>

</body>
</html>
<?php
// 1. إدارة الجلسة والأمان
session_start();
if (!isset($_SESSION['user_id'])) {
    header("Location: login.php");
    exit();
}

// 2. الاتصال بقاعدة البيانات
require_once('config/database.php');

// 3. جلب بيانات الموكلين
try {
    $stmt = $pdo->query("SELECT * FROM clients ORDER BY created_at DESC");
    $clients = $stmt->fetchAll();
} catch (Exception $e) {
    $clients = [];
}
?>

<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <title>إدارة الموكلين - نظام العدالة</title>
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.rtl.min.css">
    <link href="https://fonts.googleapis.com/css2?family=Tajawal:wght@300;500;800&display=swap" rel="stylesheet">
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    
    <style>
        :root {
            --law-dark: #1a237e;
            --law-gold: #c5a059;
        }

        body {
            background-color: #f0f2f5;
            font-family: 'Tajawal', sans-serif;
            display: flex;
        }

        /* القائمة الجانبية المستمرة */
        .sidebar {
            width: 280px;
            background: var(--law-dark);
            min-height: 100vh;
            color: white;
            position: fixed;
        }

        .nav-link {
            color: rgba(255,255,255,0.8);
            padding: 15px 25px;
            display: block;
            text-decoration: none;
            transition: 0.3s;
        }

        .nav-link:hover, .nav-link.active {
            background: rgba(255,255,255,0.05);
            color: var(--law-gold);
            border-right: 4px solid var(--law-gold);
        }

        /* محتوى الصفحة */
        .main-content {
            margin-right: 280px;
            width: 100%;
            padding: 40px;
        }

        .client-table-card {
            background: white;
            border-radius: 20px;
            padding: 30px;
            box-shadow: 0 10px 30px rgba(0,0,0,0.05);
            border: none;
        }

        .table thead th {
            background-color: #f8f9fa;
            color: var(--law-dark);
            border-bottom: 2px solid var(--law-gold);
            padding: 15px;
        }

        .table tbody td {
            padding: 15px;
            vertical-align: middle;
            border-bottom: 1px solid #eee;
        }

        .avatar-circle {
            width: 40px;
            height: 40px;
            background: var(--law-gold);
            color: white;
            border-radius: 50%;
            display: flex;
            align-items: center;
            justify-content: center;
            font-weight: bold;
            margin-left: 10px;
        }

        .btn-action {
            width: 35px;
            height: 35px;
            border-radius: 8px;
            display: inline-flex;
            align-items: center;
            justify-content: center;
            margin: 0 2px;
            transition: 0.3s;
        }
    </style>
</head>
<body>

    <div class="sidebar text-center py-4">
        <i class="fas fa-balance-scale fa-3x text-warning mb-3"></i>
        <h4 class="mb-5">نظام العدالة</h4>
        <div class="text-end">
            <a href="index.php" class="nav-link"><i class="fas fa-th-large ms-2"></i> لوحة التحكم</a>
            <a href="clients.php" class="nav-link active"><i class="fas fa-users ms-2"></i> الموكلين</a>
            <a href="cases.php" class="nav-link"><i class="fas fa-briefcase ms-2"></i> القضايا</a>
            <a href="finance.php" class="nav-link"><i class="fas fa-wallet ms-2"></i> الحسابات</a>
            <a href="logout.php" class="nav-link text-danger mt-5"><i class="fas fa-power-off ms-2"></i> خروج</a>
        </div>
    </div>

    <div class="main-content">
        <div class="d-flex justify-content-between align-items-center mb-4">
            <div>
                <h2 class="fw-bold text-dark">دليل الموكلين</h2>
                <p class="text-muted">إدارة بيانات العملاء والبحث في السجلات</p>
            </div>
            <a href="add_client.php" class="btn btn-dark px-4 py-2 rounded-pill shadow">
                <i class="fas fa-user-plus me-2"></i> إضافة موكل جديد
            </a>
        </div>

        <div class="client-table-card">
            <div class="table-responsive">
                <table class="table">
                    <thead>
                        <tr>
                            <th>الاسم الكامل</th>
                            <th>رقم الهاتف</th>
                            <th>البريد الإلكتروني</th>
                            <th>تاريخ التسجيل</th>
                            <th>الإجراءات</th>
                        </tr>
                    </thead>
                    <tbody>
                        <?php if (empty($clients)): ?>
                            <tr><td colspan="5" class="text-center py-5 text-muted">لا يوجد موكلين مسجلين حالياً</td></tr>
                        <?php else: ?>
                            <?php foreach ($clients as $client): ?>
                            <tr>
                                <td>
                                    <div class="d-flex align-items-center">
                                        <div class="avatar-circle">
                                            <?php echo mb_substr($client['name'], 0, 1, 'utf-8'); ?>
                                        </div>
                                        <span class="fw-bold"><?php echo htmlspecialchars($client['name']); ?></span>
                                    </div>
                                </td>
                                <td><i class="fas fa-phone-alt me-2 text-muted small"></i> <?php echo htmlspecialchars($client['phone']); ?></td>
                                <td><?php echo htmlspecialchars($client['email']); ?></td>
                                <td><?php echo date('Y/m/d', strtotime($client['created_at'])); ?></td>
                                <td>
                                    <a href="edit_client.php?id=<?php echo $client['id']; ?>" class="btn-action bg-primary bg-opacity-10 text-primary" title="تعديل">
                                        <i class="fas fa-edit"></i>
                                    </a>
                                    <a href="client_details.php?id=<?php echo $client['id']; ?>" class="btn-action bg-info bg-opacity-10 text-info" title="تفاصيل">
                                        <i class="fas fa-eye"></i>
                                    </a>
                                    <a href="delete_client.php?id=<?php echo $client['id']; ?>" class="btn-action bg-danger bg-opacity-10 text-danger" onclick="return confirm('هل أنت متأكد من الحذف؟')" title="حذف">
                                        <i class="fas fa-trash"></i>
                                    </a>
                                </td>
                            </tr>
                            <?php endforeach; ?>
                        <?php endif; ?>
                    </tbody>
                </table>
            </div>
        </div>
    </div>

</body>
</html>
<?php
session_start();
if (!isset($_SESSION['user_id'])) {
    header("Location: login.php");
    exit();
}

require_once('config/database.php');

try {
    // 1. إحصائيات القضايا
    $total_cases = $pdo->query("SELECT COUNT(*) FROM cases")->fetchColumn();
    $active_cases = $pdo->query("SELECT COUNT(*) FROM cases WHERE status = 'متداولة'")->fetchColumn();
    
    // 2. إحصائيات الموكلين
    $total_clients = $pdo->query("SELECT COUNT(*) FROM clients")->fetchColumn();

    // 3. الإحصائيات المالية (أرباح الشهر الحالي)
    $current_month = date('m');
    $current_year = date('Y');
    
    // إجمالي المقبوضات هذا الشهر
    $stmt_income = $pdo->prepare("SELECT SUM(amount) FROM transactions WHERE type = 'payment' AND MONTH(date) = ? AND YEAR(date) = ?");
    $stmt_income->execute([$current_month, $current_year]);
    $monthly_income = $stmt_income->fetchColumn() ?? 0;

    // إجمالي المصروفات هذا الشهر
    $stmt_expenses = $pdo->prepare("SELECT SUM(amount) FROM transactions WHERE type = 'expense' AND MONTH(date) = ? AND YEAR(date) = ?");
    $stmt_expenses->execute([$current_month, $current_year]);
    $monthly_expenses = $stmt_expenses->fetchColumn() ?? 0;

    // 4. المهام العاجلة
    $urgent_tasks = $pdo->query("SELECT COUNT(*) FROM tasks WHERE status = 'pending' AND priority = 'high'")->fetchColumn();

} catch (Exception $e) {
    die("خطأ في جلب الإحصائيات: " . $e->getMessage());
}
?>

<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <title>لوحة التحكم | الإحصائيات العامة</title>
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.rtl.min.css">
    <link href="https://fonts.googleapis.com/css2?family=Tajawal:wght@400;700&display=swap" rel="stylesheet">
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <style>
        body { font-family: 'Tajawal', sans-serif; background-color: #f0f2f5; }
        .stat-card { border: none; border-radius: 20px; transition: 0.3s; color: white; overflow: hidden; position: relative; }
        .stat-card:hover { transform: translateY(-5px); }
        .stat-card i { position: absolute; left: 10px; bottom: 10px; font-size: 4rem; opacity: 0.2; }
        .bg-gradient-blue { background: linear-gradient(45deg, #1a237e, #3949ab); }
        .bg-gradient-green { background: linear-gradient(45deg, #27ae60, #2ecc71); }
        .bg-gradient-orange { background: linear-gradient(45deg, #e67e22, #f39c12); }
        .bg-gradient-red { background: linear-gradient(45deg, #c0392b, #e74c3c); }
    </style>
</head>
<body class="p-4">

<div class="container">
    <div class="d-flex justify-content-between align-items-center mb-5">
        <h2 class="fw-bold text-dark"><i class="fas fa-chart-line me-2 text-primary"></i> ملخص أداء المكتب - <?php echo date('M Y'); ?></h2>
        <a href="index.php" class="btn btn-outline-dark rounded-pill">العودة للرئيسية</a>
    </div>

    <div class="row g-4">
        <div class="col-md-3">
            <div class="card stat-card bg-gradient-blue p-4 shadow">
                <h6>إجمالي القضايا</h6>
                <h2 class="fw-bold"><?php echo $total_cases; ?></h2>
                <p class="mb-0 small">منها <?php echo $active_cases; ?> متداولة حالياً</p>
                <i class="fas fa-gavel"></i>
            </div>
        </div>

        <div class="col-md-3">
            <div class="card stat-card bg-gradient-orange p-4 shadow">
                <h6>عدد الموكلين</h6>
                <h2 class="fw-bold"><?php echo $total_clients; ?></h2>
                <p class="mb-0 small">موكل مسجل في النظام</p>
                <i class="fas fa-users"></i>
            </div>
        </div>

        <div class="col-md-3">
            <div class="card stat-card bg-gradient-green p-4 shadow">
                <h6>دخل الشهر الحالي</h6>
                <h2 class="fw-bold"><?php echo number_format($monthly_income, 0); ?> ج.م</h2>
                <p class="mb-0 small">صافي الربح: <?php echo number_format($monthly_income - $monthly_expenses, 0); ?></p>
                <i class="fas fa-money-bill-wave"></i>
            </div>
        </div>

        <div class="col-md-3">
            <div class="card stat-card bg-gradient-red p-4 shadow">
                <h6>مهام عاجلة جداً</h6>
                <h2 class="fw-bold"><?php echo $urgent_tasks; ?></h2>
                <p class="mb-0 small">يجب إنجازها فوراً</p>
                <i class="fas fa-exclamation-circle"></i>
            </div>
        </div>
    </div>

    <div class="row mt-5">
        <div class="col-md-12">
            <div class="card p-4 border-0 shadow-sm rounded-4">
                <h5 class="fw-bold mb-3">إجراءات سريعة</h5>
                <div class="d-flex gap-2">
                    <a href="add_case.php" class="btn btn-primary px-4"><i class="fas fa-folder-plus me-2"></i> إضافة قضية</a>
                    <a href="tasks.php" class="btn btn-dark px-4"><i class="fas fa-tasks me-2"></i> قائمة المهام</a>
                    <a href="clients.php" class="btn btn-light border px-4"><i class="fas fa-user-plus me-2"></i> إضافة موكل</a>
                </div>
            </div>
        </div>
    </div>
</div>

</body>
</html>
<?php
session_start();
if (!isset($_SESSION['user_id'])) {
    header("Location: login.php");
    exit();
}

require_once('config/database.php');

// 1. جلب معرف القضية المطلوب تعديلها
if (!isset($_GET['id']) || empty($_GET['id'])) {
    header("Location: cases.php");
    exit();
}

$case_id = $_GET['id'];
$message = "";

// 2. معالجة طلب التحديث عند إرسال النموذج
if ($_SERVER['REQUEST_METHOD'] == 'POST') {
    $case_number = $_POST['case_number'];
    $case_type   = $_POST['case_type'];
    $status      = $_POST['status'];
    $client_id   = $_POST['client_id'];

    try {
        $sql = "UPDATE cases SET case_number = ?, case_type = ?, status = ?, client_id = ? WHERE id = ?";
        $stmt = $pdo->prepare($sql);
        if ($stmt->execute([$case_number, $case_type, $status, $client_id, $case_id])) {
            $message = "<div class='alert alert-success shadow-sm'>تم تحديث بيانات القضية بنجاح! <a href='view_case.php?id=$case_id'>عرض التغييرات</a></div>";
        }
    } catch (Exception $e) {
        $message = "<div class='alert alert-danger shadow-sm'>خطأ أثناء التحديث: " . $e->getMessage() . "</div>";
    }
}

// 3. جلب بيانات القضية الحالية لعرضها في النموذج
$stmt_case = $pdo->prepare("SELECT * FROM cases WHERE id = ?");
$stmt_case->execute([$case_id]);
$case = $stmt_case->fetch();

if (!$case) {
    die("القضية غير موجودة.");
}

// 4. جلب قائمة الموكلين لملء القائمة المنسدلة
$stmt_clients = $pdo->query("SELECT id, name FROM clients ORDER BY name ASC");
$clients = $stmt_clients->fetchAll();
?>

<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>تعديل قضية | <?php echo htmlspecialchars($case['case_number']); ?></title>
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.rtl.min.css">
    <link href="https://fonts.googleapis.com/css2?family=Tajawal:wght@400;700&display=swap" rel="stylesheet">
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <style>
        body { font-family: 'Tajawal', sans-serif; background-color: #f0f2f5; padding-top: 40px; }
        .edit-card { border: none; border-radius: 20px; box-shadow: 0 10px 30px rgba(0,0,0,0.08); overflow: hidden; }
        .card-header-edit { background: #1a237e; color: white; padding: 25px; text-align: center; border-bottom: 5px solid #c5a059; }
        .btn-update { background-color: #1a237e; color: white; font-weight: bold; border-radius: 10px; padding: 12px; transition: 0.3s; }
        .btn-update:hover { background-color: #c5a059; color: white; }
        .form-label { color: #1a237e; font-weight: bold; }
    </style>
</head>
<body>

<div class="container mb-5">
    <div class="row justify-content-center">
        <div class="col-md-7">
            <div class="card edit-card">
                <div class="card-header-edit">
                    <i class="fas fa-edit fa-2x mb-2" style="color: #c5a059;"></i>
                    <h3 class="fw-bold m-0">تعديل بيانات القضية</h3>
                </div>

                <div class="card-body p-4 p-md-5 bg-white">
                    <?php echo $message; ?>

                    <form method="POST">
                        <div class="row">
                            <div class="col-md-6 mb-3">
                                <label class="form-label">رقم القضية</label>
                                <input type="text" name="case_number" class="form-control" value="<?php echo htmlspecialchars($case['case_number']); ?>" required>
                            </div>

                            <div class="col-md-6 mb-3">
                                <label class="form-label">نوع القضية</label>
                                <input type="text" name="case_type" class="form-control" value="<?php echo htmlspecialchars($case['case_type']); ?>" required>
                            </div>
                        </div>

                        <div class="mb-3">
                            <label class="form-label">الموكل المرتبط</label>
                            <select name="client_id" class="form-select" required>
                                <?php foreach ($clients as $client): ?>
                                    <option value="<?php echo $client['id']; ?>" <?php echo ($client['id'] == $case['client_id']) ? 'selected' : ''; ?>>
                                        <?php echo htmlspecialchars($client['name']); ?>
                                    </option>
                                <?php endforeach; ?>
                            </select>
                        </div>

                        <div class="mb-4">
                            <label class="form-label">حالة القضية الحالية</label>
                            <select name="status" class="form-select">
                                <option value="متداولة" <?php echo ($case['status'] == 'متداولة') ? 'selected' : ''; ?>>متداولة</option>
                                <option value="محجوزة للحكم" <?php echo ($case['status'] == 'محجوزة للحكم') ? 'selected' : ''; ?>>محجوزة للحكم</option>
                                <option value="منتهية" <?php echo ($case['status'] == 'منتهية') ? 'selected' : ''; ?>>منتهية</option>
                                <option value="موقوفة" <?php echo ($case['status'] == 'موقوفة') ? 'selected' : ''; ?>>موقوفة</option>
                            </select>
                        </div>

                        <div class="d-grid gap-2">
                            <button type="submit" class="btn btn-update shadow-sm">
                                <i class="fas fa-save me-2"></i> حفظ التعديلات الجديدة
                            </button>
                            <a href="view_case.php?id=<?php echo $case_id; ?>" class="btn btn-light text-muted">
                                إلغاء التعديل والعودة
                            </a>
                        </div>
                    </form>
                </div>
            </div>
        </div>
    </div>
</div>

<script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/js/bootstrap.bundle.min.js"></script>
</body>
</html>
<?php
session_start();
if (!isset($_SESSION['user_id'])) { header("Location: login.php"); exit(); }
require_once('config/database.php');

// جلب ملخص مالي لكل قضية
$query = "SELECT 
            c.id, c.case_number, cl.name as client_name,
            COALESCE(cf.total_fee, 0) as total_amount,
            COALESCE(SUM(p.amount_paid), 0) as paid_amount
          FROM cases c
          JOIN clients cl ON c.client_id = cl.id
          LEFT JOIN case_finances cf ON c.id = cf.case_id
          LEFT JOIN payments p ON c.id = p.case_id
          GROUP BY c.id";

$finances = $pdo->query($query)->fetchAll();
?>

<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <title>السجلات المالية - نظام العدالة</title>
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.rtl.min.css">
    <link href="https://fonts.googleapis.com/css2?family=Tajawal:wght@400;700&display=swap" rel="stylesheet">
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <style>
        body { font-family: 'Tajawal', sans-serif; background-color: #f8f9fa; }
        .main-content { margin-right: 280px; padding: 40px; }
        .sidebar { width: 280px; background: #1a237e; min-height: 100vh; position: fixed; color: white; }
        .nav-link { color: white; padding: 15px 25px; display: block; text-decoration: none; }
        .nav-link.active { background: #c5a059; }
        .card-finance { border: none; border-radius: 15px; box-shadow: 0 4px 12px rgba(0,0,0,0.05); }
    </style>
</head>
<body>

    <div class="sidebar">
        <div class="p-4 text-center"><h4>نظام العدالة</h4></div>
        <a href="index.php" class="nav-link">لوحة التحكم</a>
        <a href="finance.php" class="nav-link active">الحسابات المالية</a>
    </div>

    <div class="main-content">
        <div class="d-flex justify-content-between align-items-center mb-4">
            <h2 class="fw-bold">الإدارة المالية للأتعاب 💰</h2>
            <button class="btn btn-success" data-bs-toggle="modal" data-bs-target="#paymentModal">
                <i class="fas fa-plus me-2"></i> تسجيل دفعة جديدة
            </button>
        </div>

        <div class="card card-finance p-4 bg-white">
            <table class="table table-hover align-middle text-center">
                <thead class="table-dark">
                    <tr>
                        <th>رقم القضية</th>
                        <th>الموكل</th>
                        <th>إجمالي الأتعاب</th>
                        <th>المدفوع</th>
                        <th>المتبقي</th>
                        <th>الحالة</th>
                    </tr>
                </thead>
                <tbody>
                    <?php foreach ($finances as $row): 
                        $remaining = $row['total_amount'] - $row['paid_amount'];
                        $status_color = ($remaining <= 0 && $row['total_amount'] > 0) ? 'text-success' : 'text-danger';
                    ?>
                    <tr>
                        <td class="fw-bold">#<?php echo $row['case_number']; ?></td>
                        <td><?php echo $row['client_name']; ?></td>
                        <td><?php echo number_format($row['total_amount'], 2); ?></td>
                        <td class="text-success"><?php echo number_format($row['paid_amount'], 2); ?></td>
                        <td class="<?php echo $status_color; ?> fw-bold"><?php echo number_format($remaining, 2); ?></td>
                        <td>
                            <?php if($remaining <= 0 && $row['total_amount'] > 0): ?>
                                <span class="badge bg-success">خالص</span>
                            <?php else: ?>
                                <span class="badge bg-warning text-dark">تحت التحصيل</span>
                            <?php endif; ?>
                        </td>
                    </tr>
                    <?php endforeach; ?>
                </tbody>
            </table>
        </div>
    </div>

    </body>
</html>
<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <title> البطولة - النسخة الاحترافية</title>
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <link href="https://fonts.googleapis.com/css2?family=Cairo:wght@400;600;800&display=swap" rel="stylesheet">
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">

    <style>
        :root {
            --main: #0a7c3a;
            --dark: #064a23;
            --accent: #ffd700;
            --bg: #f8fafc;
            --card: #ffffff;
            --text: #1e293b;
            --orange: #e67e22;
        }

        * { box-sizing: border-box; transition: all 0.3s ease; }

        body {
            margin: 0;
            font-family: 'Cairo', sans-serif;
            background: var(--bg);
            color: var(--text);
            overflow-x: hidden;
        }

        header {
            background: linear-gradient(135deg, var(--main), var(--dark));
            color: #fff;
            padding: 60px 20px 80px;
            text-align: center;
            position: relative;
            box-shadow: 0 4px 20px rgba(0,0,0,0.1);
            clip-path: polygon(0 0, 100% 0, 100% 85%, 0% 100%);
        }

        .lang-switch { position: absolute; left: 20px; top: 20px; z-index: 10; }
        .lang-btn { background: rgba(255,255,255,0.1); backdrop-filter: blur(5px); border: 1px solid rgba(255,255,255,0.3); color: #fff; padding: 6px 18px; border-radius: 20px; cursor: pointer; font-family: 'Cairo'; font-weight: 600; }
        .lang-btn:hover { background: var(--accent); color: var(--dark); }

        .teams-ticker {
            background: #fff;
            border-bottom: 2px solid var(--accent);
            padding: 10px 0;
            overflow: hidden;
            display: flex;
        }
        .ticker-wrapper {
            display: flex;
            animation: ticker 30s linear infinite;
            width: fit-content;
        }
        .ticker-item {
            display: flex;
            align-items: center;
            padding: 0 30px;
            font-weight: 800;
            color: var(--main);
            gap: 10px;
            white-space: nowrap;
        }
        .ticker-item img { height: 35px; width: 35px; object-fit: contain; border-radius: 50%; border: 1px solid #ddd; background: #f9f9f9; }

        /* تصحيح اتجاه الحركة ليتناسب مع RTL و LTR */
        @keyframes ticker {
            0% { transform: translateX(0); }
            100% { transform: translateX(50%); }
        }
        [dir="ltr"] @keyframes ticker {
            0% { transform: translateX(0); }
            100% { transform: translateX(-50%); }
        }

        .quick-stats {
            display: flex;
            justify-content: center;
            gap: 20px;
            margin-top: -50px;
            padding: 0 10px;
            z-index: 100;
            position: relative;
        }

        .stat-item {
            background: #fff;
            padding: 15px 30px;
            border-radius: 20px;
            box-shadow: 0 10px 30px rgba(0,0,0,0.08);
            font-weight: 700;
            display: flex;
            flex-direction: column;
            align-items: center;
            min-width: 140px;
            border: 1px solid #e2e8f0;
        }

        .stat-item i { font-size: 24px; color: var(--main); margin-bottom: 5px; }
        .stat-item b { font-size: 24px; }
        .stat-item span { font-size: 14px; color: #64748b; }

        nav {
            display: grid;
            grid-template-columns: repeat(auto-fill, minmax(220px, 1fr));
            gap: 25px;
            padding: 60px 20px;
            max-width: 1300px;
            margin: 0 auto;
        }

        .card {
            background: var(--card);
            border-radius: 24px;
            padding: 40px 20px;
            text-align: center;
            box-shadow: 0 4px 6px -1px rgba(0,0,0,0.02), 0 10px 15px -3px rgba(0,0,0,0.05);
            cursor: pointer;
            border: 1px solid #f1f5f9;
            position: relative;
            overflow: hidden;
            text-decoration: none;
            display: block;
        }

        .card:hover { 
            transform: translateY(-12px); 
            box-shadow: 0 20px 40px rgba(0,0,0,0.1); 
            border-color: var(--main);
        }

        .card i { 
            font-size: 48px; 
            background: linear-gradient(45deg, var(--main), #2ecc71);
            -webkit-background-clip: text;
            -webkit-text-fill-color: transparent;
            margin-bottom: 20px; 
        }

        .card h3 { margin: 0; font-size: 18px; font-weight: 800; color: #334155; }

        .card.special { border-bottom: 5px solid var(--orange); }
        .card.special i { background: linear-gradient(45deg, var(--orange), #f39c12); -webkit-background-clip: text; -webkit-text-fill-color: transparent; }

        .card.live { background: #fff; border: 2px solid #ef4444; }
        .card.live i { background: linear-gradient(45deg, #ef4444, #ff7675); -webkit-background-clip: text; -webkit-text-fill-color: transparent; }
        .dot { height: 10px; width: 10px; background: #ef4444; border-radius: 50%; display: inline-block; margin-left: 8px; animation: blink 1s infinite; }

        @keyframes blink { 0% {opacity:1} 50% {opacity:0.3} 100% {opacity:1} }

        footer { text-align: center; padding: 40px; color: #64748b; font-size: 14px; background: #f8fafc; border-top: 1px solid #e2e8f0; }

        @media (max-width: 600px) {
            nav { grid-template-columns: repeat(2, 1fr); gap: 15px; }
            .quick-stats { flex-wrap: wrap; margin-top: -60px; }
            .stat-item { min-width: 45%; }
            header h1 { font-size: 22px; }
        }
    </style>
</head>
<body>

<div class="teams-ticker">
    <div class="ticker-wrapper" id="teamsTicker"></div>
</div>

<header>
    <div class="lang-switch">
        <button class="lang-btn" onclick="toggleLang()">Français</button>
    </div>
    <h1 id="main-title" data-ar="🏆 نظام إدارة بطولة كرة القدم" data-fr="🏆 Système de Gestion de Football">🏆  بطولة كرة القدم</h1>
    <p id="sub-title" data-ar="لوحة التحكم الاحترافية للموسم الرياضي 2025/2026" data-fr="Tableau de bord professionnel Saison 2025/2026"> لوحة التحكم الاحترافية للموسم الرياضي 2025/2026</p>
</header>

<div class="quick-stats">
    <div class="stat-item">
        <i class="fas fa-shield-halved"></i> 
        <b id="dashTeams">0</b>
        <span data-ar="فريقاً مسجلاً" data-fr="Équipes Engagées">فريقاً مسجلاً</span>
    </div>
    <div class="stat-item">
        <i class="fas fa-users"></i> 
        <b id="dashPlayers">0</b>
        <span data-ar="لاعباً مقيداً" data-fr="Joueurs Inscrits">لاعباً مقيداً</span>
    </div>
    <div class="stat-item">
        <i class="fas fa-calendar-check"></i> 
        <b id="dashTodayMatches">0</b>
        <span data-ar="مباريات اليوم" data-fr="Matchs du Jour">مباريات اليوم</span>
    </div>
</div>

<nav>
    <div class="card" onclick="go('pages/teams.html')">
        <i class="fas fa-shield-halved"></i><h3 data-ar="إدارة الفرق" data-fr="Équipes">إدارة الفرق</h3>
    </div>
    <div class="card" onclick="go('pages/players.html')">
        <i class="fas fa-user-ninja"></i><h3 data-ar="إدارة اللاعبين" data-fr="Joueurs">إدارة اللاعبين</h3>
    </div>
    <div class="card" onclick="go('pages/school_verify.html')">
        <i class="fas fa-user-graduate"></i><h3 data-ar="مركز التحقق من الشهادات" data-fr="Vérification Scolaire">مركز التحقق من الشهادات</h3>
    </div>
    <div class="card" onclick="go('pages/matches.html')">
        <i class="fas fa-calendar-alt"></i><h3 data-ar="جدول المباريات" data-fr="Calendrier">جدول المباريات</h3>
    </div>
    <div class="card special" onclick="go('pages/results.html')">
        <i class="fas fa-poll-h"></i><h3 data-ar="تسجيل النتائج" data-fr="Résultats">تسجيل النتائج</h3>
    </div>
    <div class="card special" onclick="go('pages/feuille_match.html')">
        <i class="fas fa-file-signature"></i><h3 data-ar="ورقة المقابلة" data-fr="Feuille de Match">ورقة المقابلة</h3>
    </div>
    <div class="card live" onclick="go('pages/live.html')">
        <i class="fas fa-satellite-dish"></i>
        <h3><span class="dot"></span><span data-ar="بث مباشر" data-fr="LIVE">بث مباشر</span></h3>
    </div>
    <div class="card" onclick="go('pages/ranking.html')">
        <i class="fas fa-chart-line"></i><h3 data-ar="ترتيب البطولة" data-fr="Classement">ترتيب البطولة</h3>
    </div>
    <div class="card" onclick="go('pages/stats.html')">
        <i class="fas fa-chart-pie"></i><h3 data-ar="إحصائيات شاملة" data-fr="Statistiques">إحصائيات شاملة</h3>
    </div>
    <div class="card" onclick="go('pages/referees.html')">
        <i class="fas fa-stopwatch-20"></i><h3 data-ar="إدارة الحكام" data-fr="Arbitres">إدارة الحكام</h3>
    </div>
    <div class="card" onclick="go('pages/reports.html')">
        <i class="fas fa-file-pdf"></i><h3 data-ar="تقارير PDF" data-fr="Rapports">تقارير PDF</h3>
    </div>
    <div class="card" onclick="go('pages/sanctions.html')">
        <i class="fas fa-copy"></i><h3 data-ar="اللجنة التأديبية" data-fr="Sanctions">اللجنة التأديبية</h3>
    </div>
    <div class="card" onclick="go('pages/invoices.html')">
        <i class="fas fa-file-invoice-dollar"></i><h3 data-ar="المالية والفواتير" data-fr="Finances">المالية والفواتير</h3>
    </div>
    <div class="card" onclick="go('pages/request_participation.html')">
        <i class="fas fa-paper-plane"></i><h3 data-ar="طلب المشاركة" data-fr="Participation">طلب المشاركة</h3>
    </div>
    <div class="card" onclick="go('pages/rules.html')">
        <i class="fas fa-gavel"></i><h3 data-ar="اللائحة التنظيمية الرسمية" data-fr="Règlement">اللائحة التنظيمية الرسمية</h3>
    </div>
	
	<div class="card" onclick="go('pages/rules.html')">
        <i class="fas fa-gavel"></i><h3 data-ar="اللائحة التنظيمية الرسمية" data-fr="Règlement">اللائحة جريدةرسمية البطولة</h3>
    </div>
	
    <div class="card" onclick="go('pages/settings.html')">
        <i class="fas fa-cog"></i><h3 data-ar="إعدادات النظام" data-fr="Paramètres">إعدادات النظام</h3>
    </div>
</nav>

<footer>
    <p>© 2025/2026 – GESTION FOOTBALL | Professional </p>
</footer>

<script>
function go(url){ window.location.href = url; }

async function syncStats() {
    const teams = JSON.parse(localStorage.getItem('teams')) || [];
    const matches = JSON.parse(localStorage.getItem('matches')) || [];
    
    document.getElementById('dashTeams').innerText = teams.length;
    
    const todayStr = new Date().toISOString().split('T')[0];
    const todayCount = matches.filter(m => m.date === todayStr).length;
    document.getElementById('dashTodayMatches').innerText = todayCount;

    const dbRequest = indexedDB.open("ProArchiveDB", 1);
    dbRequest.onsuccess = (e) => {
        const db = e.target.result;
        if (db.objectStoreNames.contains("players")) {
            const transaction = db.transaction(["players"], "readonly");
            const store = transaction.objectStore("players");
            const countRequest = store.count();
            countRequest.onsuccess = () => {
                document.getElementById('dashPlayers').innerText = countRequest.result;
            };
        }
    };

    const ticker = document.getElementById('teamsTicker');
    if (teams.length > 0) {
        const content = teams.map(t => `
            <div class="ticker-item">
                <img src="${t.logo || 'https://via.placeholder.com/50'}" alt="${t.name}">
                <span>${t.name}</span>
            </div>
        `).join('');
        ticker.innerHTML = content + content; 
    } else {
        ticker.innerHTML = "<div class='ticker-item'>بانتظار تسجيل الفرق المشاركة...</div>";
    }
}

function toggleLang() {
    const currentLang = document.documentElement.lang;
    const newLang = currentLang === "ar" ? "fr" : "ar";
    document.documentElement.lang = newLang;
    document.documentElement.dir = newLang === "ar" ? "rtl" : "ltr";
    localStorage.setItem("lang", newLang);
    
    // تحديث النصوص التي تحمل سمات data-ar و data-fr
    document.querySelectorAll('[data-' + newLang + ']').forEach(el => {
        el.innerText = el.getAttribute('data-' + newLang);
    });
    
    document.querySelector('.lang-btn').innerText = newLang === "ar" ? "Français" : "العربية";
}

window.onload = () => {
    syncStats(); 
    const savedLang = localStorage.getItem("lang") || "ar";
    if(savedLang === "fr") {
        document.documentElement.lang = "ar"; // تهيئة مبدئية لتجنب التعارض
        toggleLang();
    }
};
</script>

</body>
</html>
<?php
// 1. بدء الجلسة والتحقق من تسجيل الدخول
session_start();
if (!isset($_SESSION['user_id'])) {
    header("Location: login.php");
    exit();
}

// 2. الاتصال بقاعدة البيانات لجلب الإحصائيات الحقيقية
require_once('config/database.php');

try {
    // جلب عدد القضايا الجارية
    $stmt_cases = $pdo->query("SELECT COUNT(*) FROM cases WHERE status = 'جارية'");
    $activeCasesCount = $stmt_cases->fetchColumn();

    // جلب عدد جلسات اليوم
    $todayDate = date('Y-m-d');
    $stmt_sessions = $pdo->prepare("SELECT COUNT(*) FROM sessions WHERE session_date = ?");
    $stmt_sessions->execute([$todayDate]);
    $todaySessionsCount = $stmt_sessions->fetchColumn();

    // جلب إجمالي الموكلين
    $stmt_clients = $pdo->query("SELECT COUNT(*) FROM clients");
    $totalClientsCount = $stmt_clients->fetchColumn();
} catch (Exception $e) {
    $activeCasesCount = 0;
    $todaySessionsCount = 0;
    $totalClientsCount = 0;
}
?>

<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>نظام العدالة لإدارة مكتب المحاماة</title>
    
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.rtl.min.css">
    <link href="https://fonts.googleapis.com/css2?family=Tajawal:wght@300;500;800&display=swap" rel="stylesheet">
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    
    <style>
        :root {
            --law-dark: #1a237e;
            --law-gold: #c5a059;
            --law-light: #f8f9fa;
            --law-text: #2c3e50;
        }

        body {
            font-family: 'Tajawal', sans-serif;
            background-color: #f0f2f5;
            color: var(--law-text);
            margin: 0;
            display: flex;
        }

        .sidebar {
            width: 280px;
            background: var(--law-dark);
            min-height: 100vh;
            color: white;
            position: fixed;
            transition: all 0.3s;
            box-shadow: 4px 0 10px rgba(0,0,0,0.1);
            z-index: 1000;
        }

        .sidebar-header {
            padding: 30px 20px;
            text-align: center;
            border-bottom: 1px solid rgba(255,255,255,0.1);
        }

        .sidebar-header i {
            font-size: 40px;
            color: var(--law-gold);
            margin-bottom: 10px;
        }

        .nav-link {
            color: rgba(255,255,255,0.8);
            padding: 15px 25px;
            display: flex;
            align-items: center;
            transition: 0.3s;
            border-right: 4px solid transparent;
            text-decoration: none;
        }

        .nav-link:hover, .nav-link.active {
            background: rgba(255,255,255,0.05);
            color: var(--law-gold);
            border-right-color: var(--law-gold);
        }

        .nav-link i {
            margin-left: 15px;
            width: 20px;
            text-align: center;
        }

        .main-wrapper {
            margin-right: 280px;
            width: 100%;
            padding: 40px;
        }

        .stat-card {
            background: white;
            border-radius: 15px;
            padding: 25px;
            box-shadow: 0 5px 15px rgba(0,0,0,0.05);
            display: flex;
            align-items: center;
            transition: transform 0.3s;
        }

        .stat-card:hover { transform: translateY(-5px); }

        .stat-icon {
            width: 60px; height: 60px;
            border-radius: 12px;
            display: flex; align-items: center; justify-content: center;
            font-size: 24px; margin-left: 20px;
        }

        .action-btn {
            background: white; border-radius: 12px; padding: 25px;
            text-align: center; text-decoration: none; color: var(--law-dark);
            font-weight: bold; display: block; box-shadow: 0 4px 6px rgba(0,0,0,0.02);
            border: 1px solid #edf2f7; transition: 0.3s;
        }

        .action-btn:hover { background: var(--law-dark); color: white !important; }

        .action-btn i { display: block; font-size: 30px; margin-bottom: 10px; color: var(--law-gold); }

        .logout-link { margin-top: 50px; color: #ff6b6b !important; }
        
        /* تجميل نتائج البحث */
        #searchResult a { border-bottom: 1px solid #eee; }
        #searchResult a:last-child { border-bottom: none; }
    </style>
</head>
<body>

    <div class="sidebar">
        <div class="sidebar-header">
            <i class="fas fa-balance-scale"></i>
            <h4 class="fw-bold m-0">نظام العدالة</h4>
            <small style="color: var(--law-gold)">مكتب المحاماة الرقمي</small>
        </div>
        
        <div class="px-3 mb-4 mt-3">
            <div class="input-group">
                <span class="input-group-text bg-transparent border-secondary text-white"><i class="fas fa-search"></i></span>
                <input type="text" id="globalSearch" class="form-control bg-dark text-white border-secondary shadow-none" placeholder="بحث سريع عن قضية أو موكل..." style="font-size: 0.8rem;">
            </div>
            <div id="searchResult" class="list-group position-absolute w-100 shadow-lg d-none" style="z-index: 1050; max-height: 300px; overflow-y: auto;"></div>
        </div>
        
        <div class="mt-2">
            <a href="index.php" class="nav-link active"><i class="fas fa-th-large"></i> لوحة التحكم</a>
            <a href="clients.php" class="nav-link"><i class="fas fa-users"></i> الموكلين</a>
            <a href="cases.php" class="nav-link"><i class="fas fa-briefcase"></i> القضايا</a>
            <a href="sessions.php" class="nav-link"><i class="fas fa-gavel"></i> الجلسات</a>
            <a href="finance.php" class="nav-link"><i class="fas fa-wallet"></i> الحسابات</a>
            <a href="archive.php" class="nav-link"><i class="fas fa-folder-open"></i> الأرشيف الرقمي</a>
            <a href="logout.php" class="nav-link logout-link"><i class="fas fa-power-off"></i> تسجيل الخروج</a>
        </div>
    </div>

    <div class="main-wrapper">
        <div class="d-flex justify-content-between align-items-center mb-5">
            <div>
                <h2 class="fw-bold">مرحباً بك، الأستاذ المحامي ⚖️</h2>
                <p class="text-muted">نظرة عامة على نشاط المكتب وآخر التحديثات</p>
            </div>
            <div class="text-muted fw-bold bg-white p-3 rounded shadow-sm">
                <i class="far fa-calendar-alt me-2 text-primary"></i> <?php echo date('Y/m/d'); ?>
            </div>
        </div>

        <div class="row g-4 mb-5">
            <div class="col-md-4">
                <div class="stat-card">
                    <div class="stat-icon bg-primary bg-opacity-10 text-primary"><i class="fas fa-balance-scale"></i></div>
                    <div>
                        <h6 class="text-muted mb-1 small">القضايا الجارية</h6>
                        <h3 class="fw-bold mb-0"><?php echo $activeCasesCount; ?></h3>
                    </div>
                </div>
            </div>
            <div class="col-md-4">
                <div class="stat-card">
                    <div class="stat-icon bg-danger bg-opacity-10 text-danger"><i class="fas fa-clock"></i></div>
                    <div>
                        <h6 class="text-muted mb-1 small">جلسات اليوم</h6>
                        <h3 class="fw-bold mb-0"><?php echo $todaySessionsCount; ?></h3>
                    </div>
                </div>
            </div>
            <div class="col-md-4">
                <div class="stat-card">
                    <div class="stat-icon bg-success bg-opacity-10 text-success"><i class="fas fa-user-tie"></i></div>
                    <div>
                        <h6 class="text-muted mb-1 small">إجمالي الموكلين</h6>
                        <h3 class="fw-bold mb-0"><?php echo $totalClientsCount; ?></h3>
                    </div>
                </div>
            </div>
        </div>

        <h5 class="fw-bold mb-4 border-bottom pb-2">إجراءات سريعة</h5>
        <div class="row g-3 mb-5">
            <div class="col-md-3"><a href="add_client.php" class="action-btn shadow-sm"><i class="fas fa-user-plus"></i> إضافة موكل جديد</a></div>
            <div class="col-md-3"><a href="add_case.php" class="action-btn shadow-sm"><i class="fas fa-file-signature"></i> فتح قضية جديدة</a></div>
            <div class="col-md-3"><a href="add_task.php" class="action-btn shadow-sm"><i class="fas fa-tasks text-warning"></i> إضافة مهمة</a></div>
            <div class="col-md-3"><a href="finance.php" class="action-btn shadow-sm"><i class="fas fa-file-invoice-dollar"></i> السجلات المالية</a></div>
        </div>

        <div class="bg-white p-4 rounded shadow-sm border-start border-4 border-danger mb-5">
            <h5 class="fw-bold mb-4 text-danger"><i class="fas fa-calendar-day me-2"></i> جلسات اليوم والمواعيد القادمة</h5>
            <div class="table-responsive">
                <table class="table table-hover">
                    <thead class="table-light">
                        <tr><th>الساعة</th><th>رقم القضية</th><th>الموكل</th><th>المتطلبات</th></tr>
                    </thead>
                    <tbody>
                        <?php
                        $sql_today = "SELECT s.*, c.case_number, cl.name FROM sessions s JOIN cases c ON s.case_id = c.id JOIN clients cl ON c.client_id = cl.id WHERE s.session_date >= CURDATE() ORDER BY s.session_date ASC, s.session_time ASC LIMIT 5";
                        $today_sessions = $pdo->query($sql_today)->fetchAll();
                        foreach ($today_sessions as $sess): ?>
                        <tr>
                            <td><span class="badge bg-primary px-3 py-2"><?php echo date('H:i', strtotime($sess['session_time'])); ?></span></td>
                            <td class="fw-bold"><?php echo htmlspecialchars($sess['case_number']); ?></td>
                            <td><?php echo htmlspecialchars($sess['name']); ?></td>
                            <td class="small text-muted"><?php echo htmlspecialchars($sess['requirements']); ?></td>
                        </tr>
                        <?php endforeach; ?>
                    </tbody>
                </table>
            </div>
        </div>
    </div>

    <script src="https://code.jquery.com/jquery-3.6.0.min.js"></script>
    <script>
    $(document).ready(function() {
        // البحث السريع عبر AJAX
        $('#globalSearch').on('keyup', function() {
            var query = $(this).val();
            if (query.length > 1) {
                $.ajax({
                    url: 'search_engine.php',
                    method: 'POST',
                    data: { query: query },
                    success: function(data) {
                        $('#searchResult').html(data).removeClass('d-none');
                    }
                });
            } else {
                $('#searchResult').addClass('d-none');
            }
        });

        // إغلاق قائمة البحث عند الضغط خارجها
        $(document).on('click', function (e) {
            if (!$(e.target).closest('#globalSearch').length) {
                $('#searchResult').addClass('d-none');
            }
        });

        // معالجة تشطيب المهام
        $('.form-check-input').on('change', function() {
            var checkbox = $(this);
            var taskId = checkbox.attr('id').replace('task', '');
            var listItem = checkbox.closest('.list-group-item');
            if (checkbox.is(':checked')) {
                $.ajax({
                    url: 'update_task_status.php',
                    method: 'POST',
                    data: { task_id: taskId },
                    success: function(response) {
                        if (response.trim() === 'success') {
                            listItem.fadeOut(500, function() { $(this).remove(); });
                        }
                    }
                });
            }
        });
    });
    </script>
</body>
</html>
-- 1. جدول المستخدمين (للدخول للنظام)
CREATE TABLE IF NOT EXISTS users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(50) NOT NULL UNIQUE,
    password VARCHAR(255) NOT NULL,
    full_name VARCHAR(100),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- 2. جدول الموكلين
CREATE TABLE IF NOT EXISTS clients (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    phone VARCHAR(20),
    email VARCHAR(100),
    id_number VARCHAR(50) UNIQUE,
    address TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- 3. جدول القضايا
CREATE TABLE IF NOT EXISTS cases (
    id INT AUTO_INCREMENT PRIMARY KEY,
    client_id INT NOT NULL,
    case_number VARCHAR(50) NOT NULL,
    case_type VARCHAR(100),
    court_name VARCHAR(255),
    status ENUM('جارية', 'منتهية', 'متوقفة') DEFAULT 'جارية',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (client_id) REFERENCES clients(id) ON DELETE CASCADE
);

-- 4. جدول الجلسات
CREATE TABLE IF NOT EXISTS sessions (
    id INT AUTO_INCREMENT PRIMARY KEY,
    case_id INT,
    session_date DATE NOT NULL,
    session_time TIME,
    requirements TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (case_id) REFERENCES cases(id) ON DELETE CASCADE
);

-- 5. جدول المالية (الأداءات)
CREATE TABLE IF NOT EXISTS finance (
    id INT AUTO_INCREMENT PRIMARY KEY,
    case_id INT,
    amount_paid DECIMAL(10, 2) NOT NULL,
    payment_date DATE NOT NULL,
    note TEXT,
    FOREIGN KEY (case_id) REFERENCES cases(id) ON DELETE CASCADE
);

-- 6. جدول الأرشيف (الأرشيف)
CREATE TABLE IF NOT EXISTS archive (
    id INT AUTO_INCREMENT PRIMARY KEY,
    case_id INT NOT NULL,
    file_name VARCHAR(255) NOT NULL,
    file_path VARCHAR(255) NOT NULL,
    file_description TEXT,
    upload_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (case_id) REFERENCES cases(id) ON DELETE CASCADE

-- 7. جدول التخزين (لتخزين)
CREATE TABLE IF NOT EXISTS tasks (
    id INT AUTO_INCREMENT PRIMARY KEY,
    task_title VARCHAR(255) NOT NULL,
    due_date DATE,
    priority ENUM('عاجل', 'عادي', 'منخفض') DEFAULT 'عادي',
    status ENUM('pending', 'completed') DEFAULT 'pending',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- 8. جدول الجلسات (الجلسات)
CREATE TABLE IF NOT EXISTS sessions (
    id INT AUTO_INCREMENT PRIMARY KEY,
    case_id INT NOT NULL,
    session_date DATE NOT NULL,
    session_time TIME NOT NULL,
    requirements TEXT, -- ماذا نحتاج لهذه الجلسة (مذكرات، شهود.. إلخ)
    session_notes TEXT, -- ماذا حدث في الجلسة (يُملأ لاحقاً)
    FOREIGN KEY (case_id) REFERENCES cases(id) ON DELETE CASCADE
);


-- جدول الأتعاب لكل قضية
CREATE TABLE IF NOT EXISTS case_finances (
    id INT AUTO_INCREMENT PRIMARY KEY,
    case_id INT NOT NULL,
    total_fee DECIMAL(10, 2) NOT NULL, -- إجمالي الأتعاب المتفق عليها
    FOREIGN KEY (case_id) REFERENCES cases(id) ON DELETE CASCADE
);

-- جدول الدفعات المستلمة
CREATE TABLE IF NOT EXISTS payments (
    id INT AUTO_INCREMENT PRIMARY KEY,
    case_id INT NOT NULL,
    amount_paid DECIMAL(10, 2) NOT NULL,
    payment_date DATE NOT NULL,
    notes TEXT,
    FOREIGN KEY (case_id) REFERENCES cases(id) ON DELETE CASCADE
);

CREATE TABLE documents (
    id INT AUTO_INCREMENT PRIMARY KEY,
    case_id INT,
    doc_title VARCHAR(255),
    file_path VARCHAR(255),
    upload_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (case_id) REFERENCES cases(id) ON DELETE CASCADE
);

CREATE TABLE tasks (
    id INT AUTO_INCREMENT PRIMARY KEY,
    task_text VARCHAR(255) NOT NULL,
    due_date DATE,
    priority ENUM('high', 'medium', 'low') DEFAULT 'medium',
    status ENUM('pending', 'completed') DEFAULT 'pending',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- إضافة مستخدم افتراضي لتتمكن من الدخول (كلمة المرور: 123456)
-- ملاحظة: في الواقع نستخدم password_hash، هذا السطر للتجربة فقط
INSERT INTO users (username, password, full_name) 
VALUES ('admin', '$2y$10$92IXUNpkjO0rOQ5byMi.Ye4oKoEa3Ro9llC/.og/at2.uheWG/igi', 'الأستاذ المحامي');
<?php
// 1. بدء الجلسة والاتصال بقاعدة البيانات
session_start();
require_once('config/database.php');

$error = "";

// 2. معالجة بيانات الدخول
if ($_SERVER['REQUEST_METHOD'] == 'POST') {
    $user = $_POST['username'];
    $pass = $_POST['password'];

    try {
        $stmt = $pdo->prepare("SELECT * FROM users WHERE username = ?");
        $stmt->execute([$user]);
        $account = $stmt->fetch();

        // التحقق من كلمة المرور (باستخدام التشفير)
        if ($account && password_verify($pass, $account['password'])) {
            $_SESSION['user_id'] = $account['id'];
            $_SESSION['user_name'] = $account['full_name'];
            header("Location: index.php"); // التوجه للوحة التحكم
            exit();
        } else {
            $error = "اسم المستخدم أو كلمة المرور غير صحيحة";
        }
    } catch (Exception $e) {
        $error = "حدث خطأ في الاتصال بالنظام";
    }
}
?>

<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <title>تسجيل الدخول | نظام العدالة الرقمي</title>
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.rtl.min.css">
    <link href="https://fonts.googleapis.com/css2?family=Tajawal:wght@300;500;900&display=swap" rel="stylesheet">
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    
    <style>
        :root {
            --gold: #ffd700;
            --dark-blue: #0f172a;
        }

        body, html {
            height: 100%;
            margin: 0;
            font-family: 'Tajawal', sans-serif;
            background-color: var(--dark-blue);
            display: flex;
            align-items: center;
            justify-content: center;
            overflow: hidden;
        }

        /* خلفية سينمائية متحركة */
        .bg-video {
            position: fixed;
            right: 0;
            bottom: 0;
            min-width: 100%;
            min-height: 100%;
            opacity: 0.3;
            z-index: -1;
            filter: blur(5px);
        }

        .login-box {
            width: 400px;
            padding: 40px;
            background: rgba(255, 255, 255, 0.07);
            backdrop-filter: blur(20px);
            border: 1px solid rgba(255, 255, 255, 0.1);
            border-radius: 30px;
            box-shadow: 0 25px 50px rgba(0,0,0,0.5);
            text-align: center;
            animation: fadeIn 1.5s ease-out;
        }

        @keyframes fadeIn {
            from { opacity: 0; transform: translateY(20px); }
            to { opacity: 1; transform: translateY(0); }
        }

        .login-logo i {
            font-size: 70px;
            color: var(--gold);
            text-shadow: 0 0 30px rgba(255, 215, 0, 0.4);
            margin-bottom: 20px;
        }

        h2 { color: white; font-weight: 900; margin-bottom: 30px; letter-spacing: 1px; }

        .form-control {
            background: rgba(255, 255, 255, 0.1);
            border: none;
            border-radius: 15px;
            padding: 15px;
            color: white;
            margin-bottom: 20px;
            text-align: center;
        }

        .form-control:focus {
            background: rgba(255, 255, 255, 0.2);
            box-shadow: 0 0 15px var(--gold);
            color: white;
        }

        .btn-login {
            background: var(--gold);
            color: var(--dark-blue);
            font-weight: 900;
            border: none;
            width: 100%;
            padding: 15px;
            border-radius: 15px;
            font-size: 18px;
            transition: 0.4s;
            margin-top: 10px;
        }

        .btn-login:hover {
            background: white;
            box-shadow: 0 0 25px white;
            transform: scale(1.02);
        }

        .footer-text {
            margin-top: 30px;
            color: rgba(255,255,255,0.5);
            font-size: 12px;
        }

        .error-msg {
            background: rgba(231, 76, 60, 0.2);
            color: #ff7675;
            padding: 10px;
            border-radius: 10px;
            margin-bottom: 20px;
            font-size: 14px;
            border: 1px solid rgba(231, 76, 60, 0.3);
        }
    </style>
</head>
<body>

    <div class="bg-video"></div>

    <div class="login-box">
        <div class="login-logo">
            <i class="fas fa-balance-scale-right"></i>
        </div>
        <h2>نظام العدالة</h2>
        <p class="text-white-50 mb-4 small">يرجى إثبات الهوية للوصول للملفات السرية</p>

        <?php if($error): ?>
            <div class="error-msg"><?php echo $error; ?></div>
        <?php endif; ?>

        <form action="" method="POST">
            <div class="input-group mb-3">
                <input type="text" name="username" class="form-control" placeholder="اسم المستخدم" required>
            </div>
            <div class="input-group mb-4">
                <input type="password" name="password" class="form-control" placeholder="كلمة المرور" required>
            </div>
            
            <button type="submit" class="btn-login shadow">دخول آمن</button>
        </form>

        <div class="footer-text">
            جميع الحقوق محفوظة &copy; مكتب المحاماة الرقمي 2026
        </div>
    </div>

</body>
</html>
<?php
require_once('config/database.php');

if (isset($_POST['query'])) {
    $search = "%" . $_POST['query'] . "%";
    $output = '';

    // البحث في الموكلين وفي القضايا (رقم القضية أو اسم الموكل)
    $sql = "SELECT 'client' as type, id, name as title, phone as sub 
            FROM clients 
            WHERE name LIKE ? 
            UNION 
            SELECT 'case' as type, id, case_number as title, case_type as sub 
            FROM cases 
            WHERE case_number LIKE ? 
            LIMIT 10";

    $stmt = $pdo->prepare($sql);
    $stmt->execute([$search, $search]);
    $results = $stmt->fetchAll();

    if ($results) {
        foreach ($results as $row) {
            $icon = ($row['type'] == 'client') ? 'fa-user text-primary' : 'fa-briefcase text-warning';
            $link = ($row['type'] == 'client') ? "view_client.php?id=" : "view_case.php?id=";
            $label = ($row['type'] == 'client') ? "موكل" : "قضية";

            $output .= '
            <a href="' . $link . $row['id'] . '" class="list-group-item list-group-item-action d-flex align-items-center py-3">
                <div class="ms-3 text-center" style="width: 40px;">
                    <i class="fas ' . $icon . ' fs-5"></i>
                </div>
                <div>
                    <div class="fw-bold text-dark">' . htmlspecialchars($row['title']) . '</div>
                    <small class="text-muted">' . $label . ' - ' . htmlspecialchars($row['sub']) . '</small>
                </div>
            </a>';
        }
    } else {
        $output .= '<div class="list-group-item text-center py-4 text-muted">لا توجد نتائج مطابقة للبحث</div>';
    }

    echo $output;
}
?>
<?php
session_start();
if (!isset($_SESSION['user_id'])) { header("Location: login.php"); exit(); }
require_once('config/database.php');

// جلب الجلسات مع معلومات القضية والموكل
$query = "SELECT s.*, c.case_number, cl.name as client_name 
          FROM sessions s 
          JOIN cases c ON s.case_id = c.id 
          JOIN clients cl ON c.client_id = cl.id 
          ORDER BY s.session_date ASC, s.session_time ASC";
$sessions = $pdo->query($query)->fetchAll();
?>

<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <title>أجندة الجلسات - نظام العدالة</title>
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.rtl.min.css">
    <link href="https://fonts.googleapis.com/css2?family=Tajawal:wght@400;700&display=swap" rel="stylesheet">
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <style>
        body { font-family: 'Tajawal', sans-serif; background-color: #f0f2f5; }
        .main-content { margin-right: 280px; padding: 40px; }
        .session-card { background: white; border-radius: 15px; border-right: 5px solid #1a237e; margin-bottom: 15px; transition: 0.3s; }
        .session-card:hover { transform: scale(1.01); }
        .sidebar { width: 280px; background: #1a237e; min-height: 100vh; position: fixed; color: white; }
        .nav-link { color: white; padding: 15px 25px; display: block; text-decoration: none; }
        .nav-link.active { background: #c5a059; }
    </style>
</head>
<body>

    <div class="sidebar">
        <div class="p-4 text-center"><h4>نظام العدالة</h4></div>
        <a href="index.php" class="nav-link">لوحة التحكم</a>
        <a href="cases.php" class="nav-link">القضايا</a>
        <a href="sessions.php" class="nav-link active">الجلسات</a>
    </div>

    <div class="main-content">
        <div class="d-flex justify-content-between align-items-center mb-4">
            <h2 class="fw-bold">أجندة الجلسات والمواعيد ⚖️</h2>
            <a href="add_session.php" class="btn btn-dark px-4 shadow-sm">
                <i class="fas fa-calendar-plus me-2"></i> جدولة جلسة جديدة
            </a>
        </div>

        <div class="row">
            <?php foreach ($sessions as $sess): 
                $isToday = ($sess['session_date'] == date('Y-m-d')) ? 'border-danger' : '';
            ?>
            <div class="col-12">
                <div class="session-card p-3 shadow-sm mb-3 <?php echo $isToday; ?>">
                    <div class="d-flex justify-content-between align-items-center">
                        <div>
                            <span class="badge bg-primary mb-2"><?php echo $sess['session_date']; ?> | <?php echo $sess['session_time']; ?></span>
                            <h5 class="fw-bold mb-1">قضية رقم: <?php echo htmlspecialchars($sess['case_number']); ?></h5>
                            <p class="mb-1 text-muted"><i class="fas fa-user me-1"></i> الموكل: <?php echo htmlspecialchars($sess['client_name']); ?></p>
                            <small class="text-danger"><i class="fas fa-exclamation-circle me-1"></i> المطلوب: <?php echo htmlspecialchars($sess['requirements']); ?></small>
                        </div>
                        <div class="text-start">
                            <a href="edit_session.php?id=<?php echo $sess['id']; ?>" class="btn btn-sm btn-outline-secondary">تعديل</a>
                            <button class="btn btn-sm btn-success">إضافة محضر الجلسة</button>
                        </div>
                    </div>
                </div>
            </div>
            <?php endforeach; ?>
            <?php if(empty($sessions)) echo "<p class='text-center py-5'>لا توجد جلسات مجدولة حالياً</p>"; ?>
        </div>
    </div>
</body>
</html>
<?php
session_start();
if (!isset($_SESSION['user_id'])) {
    header("Location: login.php");
    exit();
}

require_once('config/database.php');

// 1. معالجة إضافة مهمة جديدة
if ($_SERVER['REQUEST_METHOD'] == 'POST' && isset($_POST['add_task'])) {
    $task_text = $_POST['task_text'];
    $due_date  = $_POST['due_date'];
    $priority  = $_POST['priority'];

    $stmt = $pdo->prepare("INSERT INTO tasks (task_text, due_date, priority, status) VALUES (?, ?, ?, 'pending')");
    $stmt->execute([$task_text, $due_date, $priority]);
}

// 2. جلب المهام غير المنجزة
$stmt = $pdo->query("SELECT * FROM tasks WHERE status = 'pending' ORDER BY due_date ASC");
$tasks = $stmt->fetchAll();
?>

<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <title>إدارة المهام والواجبات اليومية</title>
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.rtl.min.css">
    <link href="https://fonts.googleapis.com/css2?family=Tajawal:wght@400;700&display=swap" rel="stylesheet">
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <style>
        body { font-family: 'Tajawal', sans-serif; background-color: #f8f9fa; }
        .task-card { border-radius: 15px; border: none; transition: 0.3s; }
        .task-card:hover { transform: translateY(-3px); box-shadow: 0 10px 20px rgba(0,0,0,0.05); }
        .priority-high { border-right: 5px solid #e74c3c; }
        .priority-medium { border-right: 5px solid #f1c40f; }
        .priority-low { border-right: 5px solid #2ecc71; }
        .done-anim { animation: fadeOut 0.5s forwards; }
        @keyframes fadeOut { from { opacity: 1; } to { opacity: 0; transform: translateX(-20px); } }
    </style>
</head>
<body>

<div class="container py-5">
    <div class="row">
        <div class="col-md-4 mb-4">
            <div class="card p-4 shadow-sm task-card">
                <h5 class="fw-bold mb-3"><i class="fas fa-plus-circle text-primary me-2"></i> إضافة مهمة جديدة</h5>
                <form method="POST">
                    <div class="mb-3">
                        <label class="form-label">ما هي المهمة؟</label>
                        <input type="text" name="task_text" class="form-control" placeholder="مثلاً: تجهيز مذكرة دفاع" required>
                    </div>
                    <div class="mb-3">
                        <label class="form-label">تاريخ الإنجاز</label>
                        <input type="date" name="due_date" class="form-control" value="<?php echo date('Y-m-d'); ?>">
                    </div>
                    <div class="mb-3">
                        <label class="form-label">الأولوية</label>
                        <select name="priority" class="form-select">
                            <option value="high">عاجلة جداً</option>
                            <option value="medium" selected>متوسطة</option>
                            <option value="low">عادية</option>
                        </select>
                    </div>
                    <button type="submit" name="add_task" class="btn btn-primary w-100 fw-bold">حفظ المهمة</button>
                </form>
            </div>
        </div>

        <div class="col-md-8">
            <h4 class="fw-bold mb-4"><i class="fas fa-list-check text-dark me-2"></i> قائمة المهام الحالية</h4>
            <div id="taskList">
                <?php foreach ($tasks as $task): ?>
                <div class="card p-3 mb-2 shadow-sm task-card priority-<?php echo $task['priority']; ?>" id="task-row-<?php echo $task['id']; ?>">
                    <div class="d-flex justify-content-between align-items-center">
                        <div class="d-flex align-items-center">
                            <input type="checkbox" class="form-check-input me-3 task-checkbox" data-id="<?php echo $task['id']; ?>" style="width: 22px; height: 22px; cursor: pointer;">
                            <div>
                                <div class="fw-bold text-dark"><?php echo htmlspecialchars($task['task_text']); ?></div>
                                <small class="text-muted"><i class="far fa-calendar-alt me-1"></i> موعد الإنجاز: <?php echo $task['due_date']; ?></small>
                            </div>
                        </div>
                        <span class="badge bg-light text-dark border">
                            <?php 
                                if($task['priority'] == 'high') echo "🔴 عاجلة";
                                elseif($task['priority'] == 'medium') echo "🟡 متوسطة";
                                else echo "🟢 عادية";
                            ?>
                        </span>
                    </div>
                </div>
                <?php endforeach; ?>
                
                <?php if(empty($tasks)): ?>
                    <div class="text-center py-5">
                        <i class="fas fa-clipboard-check fa-4x text-muted mb-3"></i>
                        <p class="text-muted">لا توجد مهام حالية.. أنت مبدع اليوم!</p>
                    </div>
                <?php endif; ?>
            </div>
        </div>
    </div>
</div>

<script src="https://code.jquery.com/jquery-3.6.0.min.js"></script>
<script>
$(document).ready(function() {
    $('.task-checkbox').on('change', function() {
        var taskId = $(this).data('id');
        var row = $('#task-row-' + taskId);
        
        if($(this).is(':checked')) {
            row.addClass('done-anim');
            $.ajax({
                url: 'update_task_status.php',
                method: 'POST',
                data: { task_id: taskId },
                success: function(response) {
                    setTimeout(function() { row.remove(); }, 500);
                }
            });
        }
    });
});
</script>
</body>
</html>
<?php
require_once('config/database.php');
if (isset($_POST['task_id'])) {
    $stmt = $pdo->prepare("UPDATE tasks SET status = 'completed' WHERE id = ?");
    $stmt->execute([$_POST['task_id']]);
    echo "success";
}
?>
<?php
session_start();
if (!isset($_SESSION['user_id'])) { header("Location: login.php"); exit(); }
require_once('config/database.php');

// 1. جلب القضايا لربط الملف بها
$cases = $pdo->query("SELECT cases.id, cases.case_number, clients.name FROM cases JOIN clients ON cases.client_id = clients.id")->fetchAll();

$message = "";

if ($_SERVER['REQUEST_METHOD'] == 'POST' && isset($_FILES['document'])) {
    $case_id = $_POST['case_id'];
    $description = $_POST['description'];
    
    // إعدادات الملف
    $target_dir = "uploads/";
    if (!file_exists($target_dir)) { mkdir($target_dir, 0777, true); } // إنشاء المجلد إن لم يوجد
    
    $file_extension = strtolower(pathinfo($_FILES["document"]["name"], PATHINFO_EXTENSION));
    $new_file_name = "case_" . $case_id . "_" . time() . "." . $file_extension;
    $target_file = $target_dir . $new_file_name;

    // التحقق من نوع الملف (PDF أو صور فقط)
    $allowed_types = ['pdf', 'jpg', 'jpeg', 'png'];
    if (in_array($file_extension, $allowed_types)) {
        if (move_uploaded_file($_FILES["document"]["tmp_name"], $target_file)) {
            try {
                $stmt = $pdo->prepare("INSERT INTO archive (case_id, file_name, file_path, file_description) VALUES (?, ?, ?, ?)");
                $stmt->execute([$case_id, $_FILES["document"]["name"], $target_file, $description]);
                $message = "<div class='alert alert-success'>تم رفع المستند بنجاح! <a href='archive.php'>عرض الأرشيف</a></div>";
            } catch (Exception $e) {
                $message = "<div class='alert alert-danger'>خطأ في قاعدة البيانات: " . $e->getMessage() . "</div>";
            }
        } else {
            $message = "<div class='alert alert-danger'>فشل رفع الملف إلى السيرفر. تأكد من صلاحيات المجلد.</div>";
        }
    } else {
        $message = "<div class='alert alert-warning'>عذراً، يسمح فقط بملفات PDF والصور.</div>";
    }
}
?>

<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <title>رفع مستند جديد - نظام العدالة</title>
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.rtl.min.css">
    <link href="https://fonts.googleapis.com/css2?family=Tajawal:wght@400;700&display=swap" rel="stylesheet">
    <style>
        body { font-family: 'Tajawal', sans-serif; background-color: #f0f2f5; }
        .main-content { margin-right: 280px; padding: 40px; }
        .upload-card { background: white; border-radius: 20px; padding: 40px; box-shadow: 0 10px 25px rgba(0,0,0,0.05); max-width: 700px; margin: auto; }
        .sidebar { width: 280px; background: #1a237e; min-height: 100vh; position: fixed; color: white; }
        .nav-link { color: white; padding: 15px 25px; display: block; text-decoration: none; }
    </style>
</head>
<body>

    <div class="sidebar">
        <div class="p-4 text-center"><h4>نظام العدالة</h4></div>
        <a href="index.php" class="nav-link">لوحة التحكم</a>
        <a href="archive.php" class="nav-link">الأرشيف الرقمي</a>
    </div>

    <div class="main-content">
        <div class="upload-card">
            <h3 class="fw-bold mb-4 text-center">رفع وثيقة للأرشيف 📤</h3>
            <?php echo $message; ?>
            <form method="POST" enctype="multipart/form-data">
                <div class="mb-3">
                    <label class="form-label fw-bold">اختر القضية</label>
                    <select name="case_id" class="form-select" required>
                        <?php foreach ($cases as $case): ?>
                            <option value="<?php echo $case['id']; ?>">قضية رقم: <?php echo $case['case_number']; ?> - (<?php echo $case['name']; ?>)</option>
                        <?php endforeach; ?>
                    </select>
                </div>
                <div class="mb-3">
                    <label class="form-label fw-bold">وصف المستند</label>
                    <input type="text" name="description" class="form-control" placeholder="مثلاً: صورة الهوية، الحكم الابتدائي..." required>
                </div>
                <div class="mb-4">
                    <label class="form-label fw-bold">اختر الملف (PDF, JPG, PNG)</label>
                    <input type="file" name="document" class="form-control" required>
                </div>
                <div class="d-grid">
                    <button type="submit" class="btn btn-primary btn-lg" style="background: #1a237e; border: none;">بدء الرفع والتحميل</button>
                </div>
            </form>
        </div>
    </div>

</body>
</html>
<?php
require_once('config/database.php');

// جلب كافة المستندات مع بيانات الموكل ورقم القضية
$sql = "SELECT d.*, c.case_number, cl.name as client_name 
        FROM documents d
        JOIN cases c ON d.case_id = c.id
        JOIN clients cl ON c.client_id = cl.id
        ORDER BY d.upload_date DESC";
$docs = $pdo->query($sql)->fetchAll();
?>

<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <title>الأرشيف الذكي - نظام العدالة</title>
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.rtl.min.css">
    <link href="https://fonts.googleapis.com/css2?family=Tajawal:wght@300;500;900&display=swap" rel="stylesheet">
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    
    <style>
        :root {
            --glass-bg: rgba(255, 255, 255, 0.85);
            --law-blue: #0f172a;
            --neon-gold: #ffd700;
        }

        body {
            background: linear-gradient(135deg, #0f172a 0%, #1e293b 100%);
            font-family: 'Tajawal', sans-serif;
            color: white;
            min-height: 100vh;
            padding: 40px 20px;
        }

        /* حاوية التصميم الزجاجي */
        .glass-panel {
            background: rgba(255, 255, 255, 0.05);
            backdrop-filter: blur(15px);
            border: 1px solid rgba(255, 255, 255, 0.1);
            border-radius: 25px;
            padding: 40px;
            box-shadow: 0 25px 50px rgba(0,0,0,0.3);
        }

        .main-header {
            text-align: center;
            margin-bottom: 50px;
        }

        .main-header i {
            font-size: 60px;
            color: var(--neon-gold);
            text-shadow: 0 0 20px rgba(255, 215, 0, 0.5);
            margin-bottom: 20px;
        }

        /* بطاقات الملفات الفريدة */
        .doc-card {
            background: rgba(255, 255, 255, 0.08);
            border-radius: 20px;
            padding: 25px;
            border: 1px solid rgba(255, 255, 255, 0.1);
            transition: all 0.4s cubic-bezier(0.175, 0.885, 0.32, 1.275);
            position: relative;
            overflow: hidden;
        }

        .doc-card:hover {
            transform: translateY(-10px);
            background: rgba(255, 255, 255, 0.15);
            border-color: var(--neon-gold);
        }

        .file-icon {
            font-size: 45px;
            color: var(--neon-gold);
            margin-bottom: 15px;
        }

        .client-tag {
            font-size: 12px;
            background: rgba(255, 215, 0, 0.2);
            color: var(--neon-gold);
            padding: 5px 12px;
            border-radius: 20px;
            display: inline-block;
            margin-bottom: 10px;
        }

        .btn-download {
            background: var(--neon-gold);
            color: #000;
            border: none;
            border-radius: 12px;
            font-weight: 900;
            width: 100%;
            margin-top: 15px;
            transition: 0.3s;
        }

        .btn-download:hover {
            background: #fff;
            box-shadow: 0 0 15px var(--neon-gold);
        }

        /* لمسة فنية لخلفية الصفحة */
        .circle-bg {
            position: fixed;
            width: 400px;
            height: 400px;
            background: radial-gradient(circle, rgba(197, 160, 89, 0.1) 0%, transparent 70%);
            top: -100px;
            right: -100px;
            z-index: -1;
        }
    </style>
</head>
<body>

<div class="circle-bg"></div>

<div class="container">
    <div class="main-header">
        <i class="fas fa-shield-halved"></i>
        <h1 class="fw-black">الأرشيف القانوني السحابي</h1>
        <p class="text-white-50">إدارة المستندات المحمية بتشفير النظام</p>
    </div>

    <div class="glass-panel">
        <div class="d-flex justify-content-between align-items-center mb-4">
            <h4 class="m-0"><i class="fas fa-layer-group me-2 text-warning"></i> الملفات المؤرشفة</h4>
            <a href="archive.php" class="btn btn-outline-light btn-sm rounded-pill px-4">+ رفع ملف جديد</a>
        </div>

        <div class="row g-4">
            <?php foreach($docs as $doc): ?>
            <div class="col-md-4">
                <div class="doc-card">
                    <span class="client-tag"><?php echo htmlspecialchars($doc['client_name']); ?></span>
                    <div class="file-icon"><i class="fas fa-file-pdf"></i></div>
                    <h5 class="fw-bold"><?php echo htmlspecialchars($doc['doc_name']); ?></h5>
                    <p class="small text-white-50 mb-1">رقم القضية: <?php echo $doc['case_number']; ?></p>
                    <p class="small text-white-50">التاريخ: <?php echo date('Y/m/d', strtotime($doc['upload_date'])); ?></p>
                    
                    <a href="uploads/<?php echo $doc['file_path']; ?>" target="_blank" class="btn btn-download btn-sm">
                        <i class="fas fa-eye me-1"></i> عرض المستند
                    </a>
                </div>
            </div>
            <?php endforeach; ?>
        </div>
    </div>
</div>

</body>
</html>
<?php
session_start();
if (!isset($_SESSION['user_id'])) {
    header("Location: login.php");
    exit();
}

require_once('config/config/database.php');

// التأكد من وجود معرف القضية في الرابط
if (!isset($_GET['id']) || empty($_GET['id'])) {
    die("خطأ: لم يتم تحديد القضية.");
}

$case_id = $_GET['id'];

try {
    // 1. جلب بيانات القضية مع بيانات الموكل
    $stmt = $pdo->prepare("SELECT cases.*, clients.name as client_name, clients.phone as client_phone 
                           FROM cases 
                           JOIN clients ON cases.client_id = clients.id 
                           WHERE cases.id = ?");
    $stmt->execute([$case_id]);
    $case = $stmt->fetch();

    if (!$case) {
        die("عذراً، القضية غير موجودة.");
    }

    // 2. جلب جلسات القضية
    $stmt_sessions = $pdo->prepare("SELECT * FROM sessions WHERE case_id = ? ORDER BY session_date DESC");
    $stmt_sessions->execute([$case_id]);
    $sessions = $stmt_sessions->fetchAll();

    // 3. جلب ملخص مالي سريع
    $stmt_finance = $pdo->prepare("SELECT 
        SUM(CASE WHEN type = 'payment' THEN amount ELSE 0 END) as total_paid,
        SUM(CASE WHEN type = 'expense' THEN amount ELSE 0 END) as total_expenses
        FROM transactions WHERE case_id = ?");
    $stmt_finance->execute([$case_id]);
    $finance = $stmt_finance->fetch();

} catch (Exception $e) {
    die("حدث خطأ في النظام: " . $e->getMessage());
}
?>

<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>تفاصيل القضية | <?php echo htmlspecialchars($case['case_number']); ?></title>
    
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.rtl.min.css">
    <link href="https://fonts.googleapis.com/css2?family=Tajawal:wght@300;500;800&display=swap" rel="stylesheet">
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    
    <style>
        :root {
            --law-dark: #1a237e;
            --law-gold: #c5a059;
            --law-light: #f8f9fa;
        }

        body {
            font-family: 'Tajawal', sans-serif;
            background-color: #f0f2f5;
            color: #2c3e50;
        }

        .case-header {
            background: var(--law-dark);
            color: white;
            padding: 30px 0;
            border-bottom: 5px solid var(--law-gold);
            margin-bottom: 30px;
        }

        .card {
            border: none;
            border-radius: 15px;
            box-shadow: 0 4px 12px rgba(0,0,0,0.05);
            margin-bottom: 20px;
        }

        .info-label {
            color: var(--law-gold);
            font-weight: bold;
            font-size: 0.85rem;
            display: block;
            margin-bottom: 2px;
        }

        .nav-pills .nav-link {
            color: var(--law-dark);
            font-weight: bold;
            border-radius: 10px;
            margin-left: 10px;
        }

        .nav-pills .nav-link.active {
            background-color: var(--law-dark);
            color: white;
        }

        .status-badge {
            padding: 8px 15px;
            border-radius: 50px;
            font-size: 0.9rem;
            background: rgba(255,255,255,0.2);
            border: 1px solid rgba(255,255,255,0.3);
        }

        .finance-card {
            border-right: 4px solid var(--law-gold);
        }
    </style>
</head>
<body>

<div class="case-header">
    <div class="container">
        <div class="row align-items-center">
            <div class="col-md-8">
                <nav aria-label="breadcrumb">
                    <ol class="breadcrumb">
                        <li class="breadcrumb-item"><a href="config/index.php" class="text-white opacity-75 text-decoration-none">الرئيسية</a></li>
                        <li class="breadcrumb-item"><a href="config/cases.php" class="text-white opacity-75 text-decoration-none">القضايا</a></li>
                        <li class="breadcrumb-item active text-white" aria-current="page">تفاصيل القضية</li>
                    </ol>
                </nav>
                <h2 class="fw-bold mb-0">قضية رقم: <?php echo htmlspecialchars($case['case_number']); ?></h2>
                <p class="mb-0 mt-2"><i class="fas fa-university me-2"></i> <?php echo htmlspecialchars($case['case_type']); ?></p>
            </div>
            <div class="col-md-4 text-md-start mt-3 mt-md-0">
                <span class="status-badge"><i class="fas fa-info-circle me-1"></i> الحالة: <?php echo htmlspecialchars($case['status']); ?></span>
            </div>
        </div>
    </div>
</div>

<div class="container">
    <div class="row">
        <div class="col-lg-4">
            <div class="card p-4">
                <h5 class="fw-bold mb-4"><i class="fas fa-user-tie text-primary me-2"></i> معلومات الموكل</h5>
                <div class="mb-3">
                    <span class="info-label">اسم الموكل</span>
                    <div class="fw-bold"><?php echo htmlspecialchars($case['client_name']); ?></div>
                </div>
                <div class="mb-3">
                    <span class="info-label">رقم الهاتف</span>
                    <div><a href="tel:<?php echo $case['client_phone']; ?>" class="text-decoration-none text-dark"><?php echo htmlspecialchars($case['client_phone']); ?></a></div>
                </div>
                <hr>
                <div class="d-grid gap-2">
                    <a href="config/edit_case.php?id=<?php echo $case['id']; ?>" class="btn btn-outline-primary"><i class="fas fa-edit me-1"></i> تعديل بيانات القضية</a>
                </div>
            </div>

            <div class="card p-4 finance-card">
                <h5 class="fw-bold mb-4"><i class="fas fa-wallet text-success me-2"></i> الملخص المالي</h5>
                <div class="d-flex justify-content-between mb-2">
                    <span class="text-muted">المبالغ المسددة:</span>
                    <span class="fw-bold text-success"><?php echo number_format($finance['total_paid'] ?? 0, 2); ?></span>
                </div>
                <div class="d-flex justify-content-between mb-2">
                    <span class="text-muted">المصروفات:</span>
                    <span class="fw-bold text-danger"><?php echo number_format($finance['total_expenses'] ?? 0, 2); ?></span>
                </div>
                <hr>
                <div class="d-grid">
                    <a href="config/case_finance.php?case_id=<?php echo $case['id']; ?>" class="btn btn-dark"><i class="fas fa-calculator me-1"></i> إدارة الحسابات بالتفصيل</a>
                </div>
            </div>
        </div>

        <div class="col-lg-8">
            <div class="card p-4">
                <ul class="nav nav-pills mb-4" id="pills-tab" role="tablist">
                    <li class="nav-item" role="presentation">
                        <button class="nav-link active" id="sessions-tab" data-bs-toggle="pill" data-bs-target="#sessions" type="button"><i class="fas fa-gavel me-1"></i> الجلسات</button>
                    </li>
                    <li class="nav-item" role="presentation">
                        <button class="nav-link" id="docs-tab" data-bs-toggle="pill" data-bs-target="#docs" type="button"><i class="fas fa-folder-open me-1"></i> المستندات والأرشيف</button>
                    </li>
                </ul>

                <div class="tab-content" id="pills-tabContent">
                    <div class="tab-pane fade show active" id="sessions" role="tabpanel">
                        <div class="d-flex justify-content-between align-items-center mb-3">
                            <h6 class="fw-bold m-0 text-primary">سجل تاريخ الجلسات</h6>
                            <a href="config/add_session.php?case_id=<?php echo $case['id']; ?>" class="btn btn-sm btn-primary">+ إضافة جلسة</a>
                        </div>
                        <div class="table-responsive">
                            <table class="table table-hover border">
                                <thead class="table-light">
                                    <tr>
                                        <th>التاريخ</th>
                                        <th>المتطلبات</th>
                                        <th>القرار</th>
                                    </tr>
                                </thead>
                                <tbody>
                                    <?php if ($sessions): foreach ($sessions as $s): ?>
                                    <tr>
                                        <td>
                                            <div class="fw-bold text-dark"><?php echo $s['session_date']; ?></div>
                                            <small class="text-muted"><?php echo $s['session_time']; ?></small>
                                        </td>
                                        <td class="small"><?php echo htmlspecialchars($s['requirements']); ?></td>
                                        <td><span class="text-danger fw-bold"><?php echo htmlspecialchars($s['judgment'] ?? 'بانتظار القرار'); ?></span></td>
                                    </tr>
                                    <?php endforeach; else: ?>
                                    <tr><td colspan="3" class="text-center py-4 text-muted">لا توجد جلسات مسجلة</td></tr>
                                    <?php endif; ?>
                                </tbody>
                            </table>
                        </div>
                    </div>

                    <div class="tab-pane fade text-center py-5" id="docs" role="tabpanel">
                        <i class="fas fa-file-pdf fa-4x text-muted mb-3"></i>
                        <h5>إدارة الأرشيف الرقمي للمستندات</h5>
                        <p class="text-muted">يمكنك رفع صور التوكيلات، العقود، والمذكرات الخاصة بهذه القضية.</p>
                        <a href="config/case_documents.php?case_id=<?php echo $case['id']; ?>" class="btn btn-warning px-5 fw-bold">الدخول للأرشيف الرقمي</a>
                    </div>
                </div>
            </div>
        </div>
    </div>
</div>

<script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/js/bootstrap.bundle.min.js"></script>
</body>
</html>
