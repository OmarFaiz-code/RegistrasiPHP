<?php
session_start();

// Inisialisasi users di session kalau belum ada
if (!isset($_SESSION['users'])) {
    $_SESSION['users'] = [
        ['nama' => 'Admin Hiker', 'email' => 'admin@hiker.com', 'password' => md5('admin123'), 'role' => 'admin'],
        ['nama' => 'Pendaki Sejati', 'email' => 'pendaki@hiker.com', 'password' => md5('123'), 'role' => 'user'],
    ];
}

// Jika sudah login, redirect ke destinasi
if (isset($_SESSION['logged_in']) && $_SESSION['logged_in'] === true) {
    header('Location: destinasi.php');
    exit;
}

$errors = [];
$old    = [];

if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    $nama             = trim($_POST['nama'] ?? '');
    $email            = trim($_POST['email'] ?? '');
    $password         = trim($_POST['password'] ?? '');
    $confirm_password = trim($_POST['confirm_password'] ?? '');

    $old = compact('nama', 'email');

    // Validasi
    if (empty($nama))                                    $errors[] = 'Nama wajib diisi.';
    if (empty($email))                                   $errors[] = 'Email wajib diisi.';
    if (!filter_var($email, FILTER_VALIDATE_EMAIL))      $errors[] = 'Format email tidak valid.';
    if (strlen($password) < 6)                           $errors[] = 'Kata sandi minimal 6 karakter.';
    if ($password !== $confirm_password)                 $errors[] = 'Konfirmasi kata sandi tidak cocok.';

    // Cek duplikat email
    foreach ($_SESSION['users'] as $u) {
        if ($u['email'] === $email) {
            $errors[] = 'Email sudah terdaftar.';
            break;
        }
    }

    if (empty($errors)) {
        $_SESSION['users'][] = [
            'nama'     => $nama,
            'email'    => $email,
            'password' => md5($password),
            'role'     => 'user',
        ];
        header('Location: login.php?registered=1');
        exit;
    }
}
?>
<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Registrasi — Hiker Best</title>
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link href="https://fonts.googleapis.com/css2?family=Playfair+Display:wght@700;900&family=Lato:wght@300;400;700&display=swap" rel="stylesheet">
    <style>
        *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }
        :root { --green-dark:#1a4731; --green-mid:#2f8f4a; --green-light:#4ab868; --cream:#f5f0e8; --gold:#c9a84c; }
        body {
            min-height: 100vh; font-family: 'Lato', sans-serif; background: var(--green-dark);
            display: flex; align-items: center; justify-content: center; position: relative; overflow: hidden;
        }
        body::before {
            content: ''; position: fixed; inset: 0;
            background: linear-gradient(180deg, #0d2a1c 0%, #1a4731 40%, #2f6b45 100%); z-index: 0;
        }
        body::after {
            content: ''; position: fixed; bottom: 0; left: 0; right: 0; height: 55vh;
            background: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 1440 400'%3E%3Cpath fill='%230d2a1c' opacity='0.6' d='M0,400 L0,250 L180,80 L360,200 L540,50 L720,180 L900,30 L1080,160 L1260,90 L1440,200 L1440,400Z'/%3E%3C/svg%3E") no-repeat bottom center / cover; z-index: 0;
        }
        .stars {
            position: fixed; inset: 0; z-index: 0;
            background-image: radial-gradient(circle, rgba(255,255,255,0.7) 1px, transparent 1px);
            background-size: 180px 180px; animation: twinkle 8s ease-in-out infinite alternate;
        }
        @keyframes twinkle { from { opacity: 0.3; } to { opacity: 0.8; } }

        .card {
            position: relative; z-index: 10; background: var(--cream); border-radius: 24px;
            padding: 44px 40px; width: min(460px, 92vw);
            box-shadow: 0 30px 70px rgba(0,0,0,0.5), 0 0 0 1px rgba(255,255,255,0.08);
            animation: up .6s cubic-bezier(.16,1,.3,1) both;
        }
        @keyframes up { from { opacity:0; transform:translateY(30px); } to { opacity:1; transform:none; } }

        .card-header {
            display: flex; align-items: center; gap: 12px; margin-bottom: 28px;
        }
        .logo-mini {
            width: 42px; height: 42px; border-radius: 10px;
            background: linear-gradient(135deg, var(--green-light), var(--gold));
            display: flex; align-items: center; justify-content: center; font-size: 20px;
        }
        .brand-name {
            font-family: 'Playfair Display', serif; font-size: 1.3rem;
            font-weight: 900; color: var(--green-dark); letter-spacing: 2px;
        }
        .back-link {
            font-size: .72rem; color: var(--green-mid); text-decoration: none;
            letter-spacing: 1px; text-transform: uppercase; display: flex;
            align-items: center; gap: 4px; margin-left: auto;
            transition: color .2s;
        }
        .back-link:hover { color: var(--green-dark); }

        .divider { height: 1px; background: rgba(0,0,0,0.08); margin-bottom: 24px; }

        .form-title {
            font-family: 'Playfair Display', serif; font-size: 1.8rem;
            color: var(--green-dark); margin-bottom: 4px;
        }
        .form-sub { font-size: .82rem; color: #888; margin-bottom: 24px; }

        .error-list {
            background: #fff0f0; border: 1px solid #ffcccc; border-left: 4px solid #e74c3c;
            border-radius: 10px; padding: 12px 16px; margin-bottom: 20px;
        }
        .error-list p { font-size: .8rem; font-weight: 700; color: #c0392b; margin-bottom: 6px; }
        .error-list ul { margin-left: 16px; }
        .error-list li { font-size: .8rem; color: #c0392b; margin-bottom: 3px; }

        .field { margin-bottom: 16px; }
        .field label {
            display: block; font-size: .68rem; font-weight: 700;
            letter-spacing: 1.5px; text-transform: uppercase;
            color: var(--green-dark); margin-bottom: 6px;
        }
        .field input {
            width: 100%; padding: 11px 15px; border: 1.5px solid #ddd; border-radius: 10px;
            font-family: 'Lato', sans-serif; font-size: .9rem; outline: none;
            transition: border-color .3s, box-shadow .3s; background: white; color: #1a1a1a;
        }
        .field input:focus {
            border-color: var(--green-mid); box-shadow: 0 0 0 3px rgba(47,143,74,.12);
        }

        .row-2 { display: grid; grid-template-columns: 1fr 1fr; gap: 14px; }

        .btn {
            width: 100%; padding: 13px;
            background: linear-gradient(135deg, var(--green-mid), var(--green-dark));
            color: white; border: none; border-radius: 12px;
            font-family: 'Lato', sans-serif; font-size: .84rem;
            font-weight: 700; letter-spacing: 2px; text-transform: uppercase;
            cursor: pointer; margin-top: 8px; transition: all .3s;
        }
        .btn:hover { transform: translateY(-2px); box-shadow: 0 8px 24px rgba(47,143,74,.4); }
        .btn:active { transform: translateY(0); }

        .login-link {
            text-align: center; margin-top: 16px;
            font-size: .8rem; color: #888;
        }
        .login-link a { color: var(--green-mid); text-decoration: none; font-weight: 700; }
        .login-link a:hover { text-decoration: underline; }
    </style>
</head>
<body>
<div class="stars"></div>
<div class="card">
    <div class="card-header">
        <div class="logo-mini">🏔️</div>
        <span class="brand-name">HIKER BEST</span>
        <a href="login.php" class="back-link">← Login</a>
    </div>
    <div class="divider"></div>

    <h2 class="form-title">Buat Akun</h2>
    <p class="form-sub">Bergabung dengan komunitas pendaki Hiker Best</p>

    <?php if (!empty($errors)): ?>
    <div class="error-list">
        <p>⚠️ Terdapat kesalahan:</p>
        <ul>
            <?php foreach ($errors as $e): ?>
                <li><?= htmlspecialchars($e) ?></li>
            <?php endforeach; ?>
        </ul>
    </div>
    <?php endif; ?>

    <form method="POST" action="register.php">
        <div class="field">
            <label>Nama Lengkap</label>
            <input type="text" name="nama" placeholder="Nama lengkap kamu"
                   value="<?= htmlspecialchars($old['nama'] ?? '') ?>" required>
        </div>
        <div class="field">
            <label>Alamat Email</label>
            <input type="email" name="email" placeholder="contoh@email.com"
                   value="<?= htmlspecialchars($old['email'] ?? '') ?>" required>
        </div>
        <div class="row-2">
            <div class="field">
                <label>Kata Sandi</label>
                <input type="password" name="password" placeholder="Min. 6 karakter" minlength="6" required>
            </div>
            <div class="field">
                <label>Konfirmasi</label>
                <input type="password" name="confirm_password" placeholder="Ulangi" required>
            </div>
        </div>
        <button type="submit" class="btn">🥾 Daftar Sekarang</button>
    </form>

    <p class="login-link">Sudah punya akun? <a href="login.php">Masuk di sini</a></p>
</div>
</body>
</html>
