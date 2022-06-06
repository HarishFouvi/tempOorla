<style>
  .items[data-value="0"] button {
    display: none;
  }
</style>
<!-- <img
      src="{{ product.images[0] | image_url }}"
      alt=""
      width="150"
      height="150"
    /> -->
<div>
  {% assign builderId =product.id %}
  <p>builderId=={{ builderId }} {{ cart.currency.iso_code }}</p>
  <button class="items-submit-btn">Build</button>
  {% for collection in product.collections %} {% for product in
  collection.products %}
  <div style="padding: 20px">
    <p>
      {{ product.title }} {{ product.variants[0].id }}
      {{ product.variants[0].weight }}{{ product.variants[0].weight_unit }}--
      {{ product.price | money }}
    </p>

    <div
      style="display: flex; max-width: 300px"
      class="items"
      data-value="0"
      data-item="{{ product.id }}"
      data-item-title="{{ product.title }}"
      data-item-pr="{{ product.price | money }}"
      data-item-unit="{{ product.variants[0].weight }}"
    >
      <span
        class="item-quantity-{{ product.id }}"
        style="width: 30; height: 30; border-radius: 50%; border: 1px solid"
      ></span>
      <button
        type="button"
        class="quantity-left-minus btn btn-danger btn-number"
        data-type="minus"
        data-field=""
        data-item="{{ product.id }}"
      >
        -
      </button>
      <div
        class="item-btn"
        style="background-color: blueviolet; color: white"
        data-item="{{ product.id }}"
      >
        <input
          type="number"
          id="quantity-{{ product.id }}"
          name="quantity"
          class="form-control item-quantity"
          value="0"
          min="1"
          max="100"
          style="width: 0; height: 0; opacity: 0"
          data-item="{{ product.id }}"
          data-grams="{{ product.variants[0].grams }}"
        />
        Add {{ product.price | money }}
      </div>

      <button
        type="button"
        class="quantity-right-plus btn btn-success btn-number"
        data-type="plus"
        data-field=""
        data-item="{{ product.id }}"
      >
        +
      </button>
    </div>
  </div>
  {% endfor %} {% endfor %}
  <form method="post" action="/cart/add" id="BYB-form">
    <input type="hidden" id="BYB-form_input" value="" />
    <input min="1" type="number" id="quantity" name="quantity" value="1" />
    <input type="submit" value="Add to cart" class="btn" />
  </form>
</div>

<script>
  console.log(location.hostname);
  window.onload = function () {
    if (window.jQuery) {
      $(document).ready(function () {
        var quantitiy = 0;
        $(".quantity-right-plus").click(function (e) {
          // Stop acting like a button
          e.preventDefault();
          incrementValue(e);
          //   // Get the field name
          //   var id = $(e.target).data("item");
          //   var quantity = parseInt($(`#quantity-${id}`).val());

          //   // If is not undefined
          //   $(`#quantity-${id}`).val(quantity + 1);
          //   $(e.target)
          //     .parent()
          //     .attr("data-value", quantity + 1);
          //   console.log($(`#quantity-${id}`).val(quantity + 1));
          // Increment
        });

        $(".quantity-left-minus").click(function (e) {
          // Stop acting like a button
          e.preventDefault();
          var id = $(e.target).data("item");
          // Get the field name
          var quantity = parseInt($(`#quantity-${id}`).val());

          if (quantity > 0) {
            $(`#quantity-${id}`).val(quantity - 1);
            $(e.target)
              .parent()
              .attr("data-value", quantity - 1);
            if (quantity > 1) {
              $(`.item-quantity-${id}`).css("display", "block");
              $(`.item-quantity-${id}`).html(quantity - 1);
            } else {
              $(`.item-quantity-${id}`).css("display", "none");
            }
          }
        });
      });
      $(".item-btn").click(function (e) {
        // Stop acting like a button
        e.preventDefault();
        incrementValue(e);
      });

      $(".items-submit-btn").click(function (e) {
        //items class queryselectorall
        var items = document.querySelectorAll(".items");
        var ids = [];
        //items foreach
        var values = {};
        items.forEach(function (item) {
          //item data-value
          var itemValue = item.getAttribute("data-value");
          //item data-item
          var itemId = item.getAttribute("data-item");
          //item data-item-title
          var itemTitle = item.getAttribute("data-item-title");
          //item data-item-pr
          var itemPrice = item.getAttribute("data-item-pr");
          //item data-item-unit
          var itemUnit = item.getAttribute("data-item-unit");
          //item data-item-quantity
          if (itemValue > 0) {
            //   values.push({
            //     id: itemId,
            //     title: itemTitle,
            //     price: itemPrice,
            //     unit: itemUnit,
            //     quantity: parseInt(itemValue),
            //   });
            ids.push(itemId);
            values[itemId] = {
              id: itemId,
              title: itemTitle,
              price: itemPrice,
              unit: itemUnit,
              quantity: parseInt(itemValue),
            };
          }

          //   console.log(itemId, itemTitle, itemPrice, itemUnit, itemValue);
        });
        console.table(ids);
        console.log(values);
        // fetch post method
        fetch("http://localhost:80/createVariant", {
          method: "POST",
          headers: {
            "Content-Type": "application/json",
          },
          body: JSON.stringify({
            productId: "{{ builderId }}",
            properties: values,
            currencyCode: "{{ cart.currency.iso_code }}",
            selectedItems: ids,
          }),
        })
          .then((response) => response.json())
          .then((data) => {
            console.log(parseInt(data.newvariantId), data.newvariantId);
            //submit the cart form
            // document.getElementById("BYB-form_input").value =
            //   JSON.stringify(data);
            // document.getElementById("BYB-form").submit();
            //shopify add to cart
            //fetch method add to cart shopify
            fetch("/cart/add.js", {
              method: "POST",
              headers: {
                "Content-Type": "application/json",
              },
              body: JSON.stringify({
                items: [
                  {
                    id: parseInt(data.newvariantId),
                    quantity: 1,
                    properties: values,
                  },
                ],
              }),
            }).then((response) => (window.location.href = "/cart"));
          });
      });
    }
  };

  function incrementValue(e) {
    var id = $(e.target).data("item");
    // Get the field name
    var quantity = parseInt($(`#quantity-${id}`).val());

    $(`#quantity-${id}`).val(quantity + 1);
    $(e.target)
      .parent()
      .attr("data-value", quantity + 1);
    $(`.item-quantity-${id}`).html(quantity + 1);
    $(`.item-quantity-${id}`).css("display", "block");
  }
</script>
