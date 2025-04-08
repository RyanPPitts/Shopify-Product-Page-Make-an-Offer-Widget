# Shopify Product Page Make an Offer Widget
 
## 🎁 Email Discount Widget – Liquid, HTML & CSS

Use this snippet to render a discount widget that collects emails and applies a 10% coupon at checkout.

```liquid
{% render 'email-discount-widget' %}


<div id="email-discount-widget" class="email-discount-widget" style="display: none;">
  <div class="discount-card">
    <p class="discount-headline"><strong>Get a 10% discount now !!</strong></p>
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


 On the main product liquid file you can insert the js file below or add as a sourced file from the assets folder
 
```javascript
<script>
  // Wait for the full page to load before running this script
  document.addEventListener("DOMContentLoaded", function () {
    // Get references to the HTML elements by their IDs
    const widget = document.getElementById("email-discount-widget");
    const button = document.getElementById("get-discount-btn");
    const input = document.getElementById("discount-email");
    const message = document.getElementById("discount-message");

    // Check if the discount was already claimed by this visitor
    if (!localStorage.getItem("discount_claimed")) {
      // If not claimed, prepare the widget to fade in
      widget.style.opacity = 0;
      widget.style.display = "block";

      // Use a short delay to trigger the fade-in animation
      setTimeout(() => {
        widget.style.transition = "opacity 0.5s ease";
        widget.style.opacity = 1;
      }, 10);

      // Automatically focus on the email input field
      input.focus();
    } else {
      // If already claimed, show a reminder message instead of the form
      document.getElementById("discount-reminder").style.display = "block";
    }

    // Listen for when the user clicks the "Get Discount" button
    button.addEventListener("click", function (event) {
      event.preventDefault(); // Prevent the form from submitting normally
      button.disabled = true; // Disable button to prevent double-clicking

      // Get and clean up the entered email address
      const email = input.value.trim();

      // Validate the email format
      if (!email || !validateEmail(email)) {
        message.innerText = "Please enter a valid email address.";
        button.disabled = false; // Re-enable the button if email is invalid
        return;
      }

      // Create a fake discount code (replace this with a real one)
      const code = "test";

      // Show a success message with the discount code
      message.innerHTML = `🎉 Your discount code is: <strong>${code}</strong><br>It will be automatically applied at checkout.`;

      // Show the reminder message
      document.getElementById("discount-reminder").style.display = "block";

      // After a 3-second delay, redirect the user to the same page with the discount code applied
      setTimeout(() => {
        const currentUrl = window.location.pathname + window.location.search;
        window.location.href = `/discount/test?redirect=${currentUrl}`;
      }, 3000);

      // Save that the user claimed the discount so the widget doesn’t show again
      localStorage.setItem("discount_claimed", "true");

      // Send the email and discount code to your server for processing
      sendEmailToUser(email, code);

      // Disable the input so the user can’t change it after submitting
      input.disabled = true;
    });

    // Helper function to check if the email is in a valid format
    function validateEmail(email) {
      return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);
    }

    // Helper function to send the email and discount code to your email service
    function sendEmailToUser(email, code) {
      fetch('https://your-email-handler.com/send', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json'
        },
        body: JSON.stringify({ email, code })
      }).catch(err => {
        // If the request fails, log a warning
        console.warn("Email send failed:", err);
      });
    }

    // Helper function to convert price in cents to dollars
    function formatMoney(cents) {
      const value = (cents / 100).toFixed(2);
      return `$${value}`;
    }

    // Safer version of the money formatter that checks for Shopify’s format function
    function safeFormatMoney(price) {
      if (typeof Shopify !== 'undefined' && typeof Shopify.formatMoney === 'function') {
        return Shopify.formatMoney(price, Shopify.money_format);
      }
      return formatMoney(price);
    }

    // Listen for variant (product option) changes on the page
    document.addEventListener("variant:change", function (event) {
      const variant = event.detail.variant;
      const priceEl = document.querySelector('.product-form__variant-price');

      // Update the price shown on the page when the variant changes
      if (variant && priceEl) {
        priceEl.textContent = safeFormatMoney(variant.price);
      }
    });
  });
</script>
