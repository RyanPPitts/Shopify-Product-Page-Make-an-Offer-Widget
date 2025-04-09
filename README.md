# Shopify Make an Offer Widget – Block Integration

This guide walks you through how to integrate a customizable "Make an Offer" widget into your Shopify product page **as a block** using the Shopify `main-product.liquid` section.

---

## 🔧 Step 1: Add the Offer Widget Block Schema to `main-product.liquid`

Add the following block configuration inside the `"blocks"` array of your `{% schema %}` tag in the `sections/main-product.liquid` file:

```json
{
  "type": "offer_widget",
  "name": "Make an Offer",
  "settings": [
    {
      "type": "text",
      "id": "button_text",
      "label": "Button text",
      "default": "💬 Make an Offer"
    },
    {
      "type": "color",
      "id": "button_color",
      "label": "Button background color",
      "default": "#000000"
    },
    {
      "type": "color",
      "id": "button_text_color",
      "label": "Button text color",
      "default": "#ffffff"
    },
    {
      "type": "color",
      "id": "form_bg",
      "label": "Form background color",
      "default": "#ffffff"
    },
    {
      "type": "text",
      "id": "widget_width",
      "label": "Widget width (e.g. 100%, 300px)",
      "default": "100%"
    },
    {
      "type": "text",
      "id": "label_offer_price",
      "label": "Label - Offer Price",
      "default": "Offer Price ($):"
    },
    {
      "type": "text",
      "id": "label_name",
      "label": "Label - Name",
      "default": "Your Name:"
    },
    {
      "type": "text",
      "id": "label_email",
      "label": "Label - Email",
      "default": "Your Email:"
    },
    {
      "type": "text",
      "id": "submit_button_text",
      "label": "Submit Button Text",
      "default": "Submit Offer"
    },
    {
      "type": "color",
      "id": "submit_button_color",
      "label": "Submit Button Background Color",
      "default": "#27ae60"
    },
    {
      "type": "color",
      "id": "submit_button_text_color",
      "label": "Submit Button Text Color",
      "default": "#ffffff"
    },
    {
      "type": "text",
      "id": "success_message",
      "label": "Success message",
      "default": "Thanks! We’ll get back to you."
    }
  ]
}
```

---

## 📥 Step 2: Render the Offer Widget Block

In your `main-product.liquid` file, locate where the blocks are rendered (e.g. inside the product form), and add this Liquid condition:

```liquid
{% for block in section.blocks %}
  {% case block.type %}
    {% when 'offer_widget' %}
      {% render 'offer-widget', block: block, product: product %}
  {% endcase %}
{% endfor %}
```

---

## 🧩 Step 3: Create the Snippet `snippets/offer-widget.liquid`

Create a new file named `offer-widget.liquid` and paste in your widget HTML, using dynamic values from `block.settings`.

Make sure your snippet uses `block.settings` for styles and text content. Example:

```liquid
<div
  id="price-offer-widget"
  style="margin-top: 1rem; background: {{ block.settings.form_bg }}; padding: 1rem; width: {{ block.settings.widget_width }};"
>
  <button
    id="open-offer-form"
    type="button"
    style="
      padding: 10px 15px;
      background: {{ block.settings.button_color }};
      color: {{ block.settings.button_text_color }};
      border: none;
      cursor: pointer;
    "
  >
    {{ block.settings.button_text }}
  </button>

  <div
    id="offer-form-popup"
    style="display:none; margin-top: 1rem; border: 1px solid #ccc; padding: 1rem;"
  >
    <form id="price-offer-form">
      <label>{{ block.settings.label_offer_price }}</label><br>
      <input type="number" name="offer_price" required style="width: 100%; margin-bottom: 10px;"><br>

      <label>{{ block.settings.label_name }}</label><br>
      <input type="text" name="name" style="width: 100%; margin-bottom: 10px;"><br>

      <label>{{ block.settings.label_email }}</label><br>
      <input type="email" name="email" required style="width: 100%; margin-bottom: 10px;"><br>

      <input type="hidden" name="product_title" value="{{ product.title }}">
      <input type="hidden" name="product_url" value="{{ shop.url }}{{ product.url }}">

      <button
        type="submit"
        style="background: {{ block.settings.submit_button_color }}; color: {{ block.settings.submit_button_text_color }}; border: none; padding: 10px 15px;"
      >
        {{ block.settings.submit_button_text }}
      </button>
    </form>
    <div id="offer-form-success" style="display:none; color: green; margin-top: 10px;">
      {{ block.settings.success_message }}
    </div>
  </div>
</div>

<script>
  document.addEventListener("DOMContentLoaded", function () {
    const offerBtn = document.getElementById("open-offer-form");
    const offerPopup = document.getElementById("offer-form-popup");
    const offerForm = document.getElementById("price-offer-form");
    const offerSuccess = document.getElementById("offer-form-success");

    if (offerBtn) {
      offerBtn.addEventListener("click", function () {
        offerPopup.style.display = offerPopup.style.display === "none" ? "block" : "none";
      });
    }

    if (offerForm) {
      offerForm.addEventListener("submit", function (e) {
        e.preventDefault();

        const formData = new FormData(offerForm);

        fetch("https://formspree.io/f/YOUR_FORM_ID", {
          method: "POST",
          body: formData,
          headers: {
            'Accept': 'application/json'
          }
        }).then(response => {
          if (response.ok) {
            offerForm.style.display = "none";
            offerSuccess.style.display = "block";
          } else {
            alert("Something went wrong. Please try again.");
          }
        });
      });
    }
  });
</script>
```

---

✅ That’s it! You now have a flexible, customizable offer widget that lives **inside the product info section** as a block, with full Theme Editor control.

