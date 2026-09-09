<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Crystal Market — Магазин</title>
    <!-- Bootstrap 5 (можно заменить на свой CSS) -->
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">
    <!-- Font Awesome -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <style>
        body { background: #f8f9fa; font-family: -apple-system, BlinkMacSystemFont, sans-serif; }
        .product-card { cursor: pointer; transition: transform 0.2s; }
        .product-card:hover { transform: translateY(-5px); }
        .product-img { width: 100%; height: 160px; object-fit: cover; border-radius: 10px 10px 0 0; }
        .screen { display: none; }
        .screen.active { display: block; }
        .back-btn { cursor: pointer; margin-bottom: 15px; }
        .payment-address { background: #e9ecef; padding: 10px; border-radius: 5px; word-break: break-all; }
        .btn-primary { background: #4CAF50; border: none; }
        .btn-primary:hover { background: #45a049; }
        .step-indicator { font-size: 0.9rem; color: #6c757d; }
    </style>
</head>
<body>

<div class="container py-4">
    <!-- ЭКРАН 1: КАТАЛОГ -->
    <div id="screen-catalog" class="screen active">
        <h3 class="mb-3">🛍️ Выберите товар</h3>
        <div id="catalog" class="row row-cols-2 g-3"></div>
    </div>

    <!-- ЭКРАН 2: ДЕТАЛИ ТОВАРА -->
    <div id="screen-product" class="screen">
        <div class="back-btn" onclick="showCatalog()"><i class="fas fa-arrow-left"></i> Назад</div>
        <div class="card">
            <img id="product-image" src="" class="card-img-top" style="max-height: 250px; object-fit: cover;">
            <div class="card-body">
                <h4 id="product-name"></h4>
                <p id="product-description" class="text-muted"></p>
                <p class="h4 text-success">€<span id="product-price"></span></p>
                <button class="btn btn-primary w-100" onclick="showDistricts()">Выбрать район</button>
            </div>
        </div>
    </div>

    <!-- ЭКРАН 3: ВЫБОР РАЙОНА -->
    <div id="screen-districts" class="screen">
        <div class="back-btn" onclick="showProduct()"><i class="fas fa-arrow-left"></i> Назад</div>
        <h4>📍 Выберите район доставки</h4>
        <div id="district-list" class="list-group"></div>
    </div>

    <!-- ЭКРАН 4: СПОСОБ ОПЛАТЫ -->
    <div id="screen-payment-methods" class="screen">
        <div class="back-btn" onclick="showDistricts()"><i class="fas fa-arrow-left"></i> Назад</div>
        <h4>💳 Способ оплаты</h4>
        <div id="payment-method-list" class="list-group"></div>
    </div>

    <!-- ЭКРАН 5: АДРЕС ДЛЯ ОПЛАТЫ -->
    <div id="screen-payment-details" class="screen">
        <div class="back-btn" onclick="showPaymentMethods()"><i class="fas fa-arrow-left"></i> Назад</div>
        <h4>💵 Оплата</h4>
        <p id="payment-summary"></p>
        <div class="payment-address" id="payment-address"></div>
        <p class="text-muted mt-2">Отправьте точную сумму и нажмите «Я оплатил».</p>
        <button class="btn btn-success w-100" onclick="confirmPayment()">✅ Я оплатил</button>
    </div>

    <!-- ЭКРАН 6: ГОТОВО -->
    <div id="screen-done" class="screen text-center">
        <i class="fas fa-check-circle text-success" style="font-size: 5rem;"></i>
        <h4>Спасибо за заказ!</h4>
        <p class="text-muted">Менеджер скоро свяжется с вами.</p>
    </div>
</div>

<script>
// ================== ДЕМО-ДАННЫЕ (замените на API позже) ==================
const products = [
    { id: 1, name: 'Кофе арабика', description: 'Свежеобжаренный кофе, 250 г. Насыщенный вкус, аромат шоколада и орехов.', price: 4.50, imageUrl: 'https://via.placeholder.com/400x250?text=Кофе' },
    { id: 2, name: 'Чай зелёный', description: 'Листовой чай, 100 г. Нежный, с травянистыми нотками.', price: 3.20, imageUrl: 'https://via.placeholder.com/400x250?text=Чай' },
    { id: 3, name: 'Шоколад ручной работы', description: 'Набор из 9 конфет с разными начинками. Премиальное качество.', price: 7.80, imageUrl: 'https://via.placeholder.com/400x250?text=Шоколад' }
];

const districts = [
    { id: 1, name: 'Центр' },
    { id: 2, name: 'Аспарухово' },
    { id: 3, name: 'Младост' }
];

const paymentMethods = {
    'USDT': '0x1111111111111111111111111111111111111111',
    'USDC Solana': '0x2222222222222222222222222222222222222222',
    'BNB': '0x3333333333333333333333333333333333333333',
    'BTC': 'bc1qxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx'
};

// ================== СОСТОЯНИЕ ==================
let selectedProduct = null;
let selectedDistrict = null;
let selectedPaymentMethod = null;

// ================== ПОКАЗ ЭКРАНОВ ==================
function showScreen(id) {
    document.querySelectorAll('.screen').forEach(s => s.classList.remove('active'));
    document.getElementById(id).classList.add('active');
}

// ================== КАТАЛОГ ==================
function renderCatalog() {
    const container = document.getElementById('catalog');
    container.innerHTML = '';
    products.forEach(p => {
        const col = document.createElement('div');
        col.className = 'col';
        col.innerHTML = `
            <div class="card product-card h-100" onclick="openProduct(${p.id})">
                <img src="${p.imageUrl}" class="product-img" alt="${p.name}">
                <div class="card-body">
                    <h6 class="card-title">${p.name}</h6>
                    <p class="card-text text-success fw-bold">€${p.price.toFixed(2)}</p>
                </div>
            </div>
        `;
        container.appendChild(col);
    });
}

function openProduct(id) {
    selectedProduct = products.find(p => p.id === id);
    document.getElementById('product-image').src = selectedProduct.imageUrl;
    document.getElementById('product-name').textContent = selectedProduct.name;
    document.getElementById('product-description').textContent = selectedProduct.description;
    document.getElementById('product-price').textContent = selectedProduct.price.toFixed(2);
    showScreen('screen-product');
}

function showCatalog() {
    showScreen('screen-catalog');
}
function showProduct() {
    showScreen('screen-product');
}

// ================== РАЙОНЫ ==================
function showDistricts() {
    const list = document.getElementById('district-list');
    list.innerHTML = '';
    districts.forEach(d => {
        list.innerHTML += `<button class="list-group-item list-group-item-action" onclick="selectDistrict(${d.id})">📍 ${d.name}</button>`;
    });
    showScreen('screen-districts');
}

function selectDistrict(id) {
    selectedDistrict = districts.find(d => d.id === id);
    showPaymentMethods();
}

// ================== СПОСОБЫ ОПЛАТЫ ==================
function showPaymentMethods() {
    const list = document.getElementById('payment-method-list');
    list.innerHTML = '';
    Object.keys(paymentMethods).forEach(method => {
        list.innerHTML += `<button class="list-group-item list-group-item-action" onclick="selectPaymentMethod('${method}')">💳 ${method}</button>`;
    });
    showScreen('screen-payment-methods');
}

function selectPaymentMethod(method) {
    selectedPaymentMethod = method;
    showPaymentDetails();
}

// ================== ДЕТАЛИ ОПЛАТЫ ==================
function showPaymentDetails() {
    const summary = `📦 ${selectedProduct.name}<br>📍 ${selectedDistrict.name}<br>💰 €${selectedProduct.price.toFixed(2)}<br>💳 ${selectedPaymentMethod}`;
    document.getElementById('payment-summary').innerHTML = summary;
    document.getElementById('payment-address').textContent = paymentMethods[selectedPaymentMethod];
    showScreen('screen-payment-details');
}

// ================== ПОДТВЕРЖДЕНИЕ ==================
function confirmPayment() {
    // Здесь будет отправка данных в бота через Telegram.WebApp.sendData
    // Пока просто показываем финальный экран
    showScreen('screen-done');
    
    // Если мини-апп открыт внутри Telegram, можно отправить данные:
    if (window.Telegram && window.Telegram.WebApp) {
        const data = JSON.stringify({
            action: 'order_paid',
            product_id: selectedProduct.id,
            district_id: selectedDistrict.id,
            payment_method: selectedPaymentMethod
        });
        window.Telegram.WebApp.sendData(data);
        // Закрыть мини-апп (опционально)
        window.Telegram.WebApp.close();
    }
}

// ================== ИНИЦИАЛИЗАЦИЯ ==================
document.addEventListener('DOMContentLoaded', () => {
    renderCatalog();

    // Подключаем Telegram Web App
    if (window.Telegram && window.Telegram.WebApp) {
        window.Telegram.WebApp.ready();
        window.Telegram.WebApp.expand();
        // Можно настроить цвет темы под Telegram
    }
});
</script>
</body>
</html>
