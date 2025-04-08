# 🛍️ Shopify Product Page – Make an Offer Widget

Boost conversions by offering a 10% discount in exchange for a customer's email. This "Make an Offer" widget appears under the Add to Cart button, encourages email capture, and applies a discount at checkout.

---

## 📦 How It Works

1. A hidden widget is revealed to the customer when the page loads.
2. The customer enters their email and clicks **Give Me My Discount**.
3. A 10% discount code appears and is applied at checkout after a short delay. Make sure to create the discount code and assign to the discount code variable in the code below.
4. The email is optionally sent to your backend (for mailing list, Klaviyo, etc.).
5. The widget is remembered using `localStorage`, so it doesn't reappear.

---

## 🧩 1. Add the Snippet to Your Theme

In your product page or a relevant section, include the widget render tag:

```liquid
{% render 'email-discount-widget' %}
```

---

## 🖼️ 2. Widget HTML

Paste this inside the `email-discount-widget.liquid` snippet file:

```html
<div id="email-discount-widget" class="email-discount-widget" style="display: none;">
  <div class="discount-card">
    <p class="discount-headline"><strong>Get a 10% discount now!</strong></p>
    <p class="discount-subtext">Enter your email to see your discount now.</p>
    <div class="discount-form">
      <input
        type="email"
        id="discount-email"
        class="discount-input"
        placeholder="Enter your email"
        required
      >
      <button type="button" id="get-discount-btn" class="discount-button">
        Give Me My Discount
      </button>
    </div>
    <p id="discount-message" class="discount-message"></p>
  </div>
</div>

<p id="discount-reminder" class="discount-reminder" style="display: none; margin-top: 1rem;">
  ✅ Don’t forget to use your 10% coupon code at checkout! Check your email or checkout now to see your discount.
</p>
```

---

## 🎨 3. Widget CSS (Inline or Asset File)

```css
#email-discount-widget {
  display: none;
  opacity: 0;
  transition: opacity 0.5s ease;
}

.email-discount-widget {
  margin: 2rem 0;
}

.discount-card {
  background-color: #f9f9f9;
  border: 1px solid #ddd;
  padding: 2rem;
  border-radius: 8px;
  max-width: 446px;
  text-align: center;
}

.discount-headline {
  font-size: 1.8rem;
  margin-bottom: 0.5rem;
}

.discount-subtext {
  font-size: 1.4rem;
  color: #555;
  margin-bottom: 1.5rem;
}

.discount-form {
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

.discount-input {
  padding: 1rem;
  font-size: 1.5rem;
  border: 1px solid #ccc;
  border-radius: 6px;
  width: 100%;
}

.discount-button {
  display: inline-flex;
  justify-content: center;
  align-items: center;
  padding: 1rem 3rem;
  border: 0;
  border-radius: var(--buttons-radius, 8px);
  background-color: rgba(var(--color-button), var(--alpha-button-background));
  color: rgb(var(--color-button-text));
  font: inherit;
  font-size: 1.5rem;
  text-decoration: none;
  cursor: pointer;
  transition: box-shadow var(--duration-short) ease;
  box-shadow: var(--buttons-shadow, none);
}

.discount-button:hover {
  box-shadow: 0 0 0 0.2rem rgba(var(--color-button), 0.2);
}

.discount-message {
  margin-top: 1rem;
  font-size: 1.4rem;
  color: green;
}

.discount-reminder {
  margin-top: 1rem;
  padding: 1rem;
  font-size: 1.4rem;
  color: #155724;
  background-color: #d4edda;
  border: 1px solid #c3e6cb;
  border-radius: 6px;
  max-width: 440px;
  text-align: center;
}
```

---

## ⚙️ 4. JavaScript Logic (Inline or as Asset File)

Place this near the bottom of your product template or as an asset reference:

```javascript
<script>
  document.addEventListener("DOMContentLoaded", function () {
    const widget = document.getElementById("email-discount-widget");
    const button = document.getElementById("get-discount-btn");
    const input = document.getElementById("discount-email");
    const message = document.getElementById("discount-message");

    if (!localStorage.getItem("discount_claimed")) {
      widget.style.opacity = 0;
      widget.style.display = "block";
      setTimeout(() => {
        widget.style.transition = "opacity 0.5s ease";
        widget.style.opacity = 1;
      }, 10);
      input.focus();
    } else {
      document.getElementById("discount-reminder").style.display = "block";
    }

    button.addEventListener("click", function (event) {
      event.preventDefault();
      button.disabled = true;

      const email = input.value.trim();

      if (!email || !validateEmail(email)) {
        message.innerText = "Please enter a valid email address.";
        button.disabled = false;
        return;
      }

      const code = "test";

      message.innerHTML = `🎉 Your discount code is: <strong>${code}</strong><br>It will be automatically applied at checkout.`;
      document.getElementById("discount-reminder").style.display = "block";

      setTimeout(() => {
        const currentUrl = window.location.pathname + window.location.search;
        window.location.href = `/discount/test?redirect=${currentUrl}`;
      }, 3000);

      localStorage.setItem("discount_claimed", "true");
      sendEmailToUser(email, code);
      input.disabled = true;
    });

    function validateEmail(email) {
      return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);
    }

    function sendEmailToUser(email, code) {
      fetch('https://your-email-handler.com/send', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json'
        },
        body: JSON.stringify({ email, code })
      }).catch(err => {
        console.warn("Email send failed:", err);
      });
    }

    function formatMoney(cents) {
      return `$${(cents / 100).toFixed(2)}`;
    }

    function safeFormatMoney(price) {
      if (typeof Shopify !== 'undefined' && typeof Shopify.formatMoney === 'function') {
        return Shopify.formatMoney(price, Shopify.money_format);
      }
      return formatMoney(price);
    }

    document.addEventListener("variant:change", function (event) {
      const variant = event.detail.variant;
      const priceEl = document.querySelector('.product-form__variant-price');
      if (variant && priceEl) {
        priceEl.textContent = safeFormatMoney(variant.price);
      }
    });
  });
</script>
```

---

## ✅ Customization Ideas

- Replace `"test"` with a **real dynamic discount code** using Shopify Scripts or API.
- Hook email submissions into **Klaviyo, Mailchimp**, or a custom backend.
- Translate the text for multilingual stores.
- Add theme editor support to control the widget style or placement.
