<script setup>
// Import Vue helpers:
// ref() is used for reactive values that can change.
// computed() is used for values calculated from other reactive data.
import { ref, computed, onMounted } from "vue";

//                                   Sorting section

// Stores the field currently used for sorting.
// Default value is "subject".
const sortBy = ref("subject");

// Stores the sorting direction.
// Default value is ascending.
const sortOrder = ref("ascending");
//                                  Add to Cart section
const cart = ref([]);

//view cart controls
const showCart = ref(false);

// Checkout details
const customerName = ref("");
const customerPhone = ref("");

// Stores the order confirmation message
// const orderMessage = ref("");
const orderPlaced = ref(false);
// inputs validations
const nameIsValid = computed(() => {
  return /^[A-Za-z ]+$/.test(customerName.value);
});

const phoneIsValid = computed(() => {
  return /^[0-9]+$/.test(customerPhone.value);
});

//                                Categories Data

// Stores the category information shown on the homepage.
// This data is static for now, so it does not need to use ref().
const categories = [
  {
    // Category name
    name: "Creative Arts",

    // Icon shown on the category card(I used my Laptop Keyboard icons)
    icon: "🎨",

    // Short description
    description: "Drawing, painting & crafts",
  },

  {
    name: "Computing & Tech",
    icon: "💻",
    description: "Coding, robotics & games",
  },

  {
    name: "Sports & Fitness",
    icon: "⚽",
    description: "Football, tennis & fitness",
  },

  {
    name: "Music",
    icon: "🎵",
    description: "Piano, guitar & singing",
  },

  {
    name: "Science",
    icon: "🔬",
    description: "Experiments & discovery",
  },

  {
    name: "Academic Support",
    icon: "📚",
    description: "Maths, English & study skills",
  },
];

//                                Lesson Data

// Stores the classes and activities (Lesson Data).

const lessons = ref([]);
// Get lessons from backend
onMounted(() => {
  fetch("http://localhost:3000/lessons")
    .then((response) => response.json())
    .then((data) => {
      lessons.value = data.map((lesson) => ({
        ...lesson,
        id: lesson._id,
        subject: lesson.topic,
        spaces: lesson.space,
      }));
    })
    .catch((error) => {
      console.error("Failed to fetch lessons:", error);
    });
});
//                                          Sorting Lesson

// computed() creates a derived value.
// Whenever sortBy or sortOrder changes,
// Vue automatically recalculates sortedLessons.
const sortedLessons = computed(() => {
  // Create a copy of the lessons array.
  // This prevents the original lessons array from being reordered directly.
  const result = [...lessons.value];

  // Sort the copied array.
  result.sort((a, b) => {
    // Read the selected property dynamically.
    // if sortBy = "subject", this becomes a.subject and b.subject.
    let first = a[sortBy.value];
    let second = b[sortBy.value];

    // Convert text values to lowercase
    // so sorting is not affected by uppercase/lowercase letters.
    if (typeof first === "string") {
      first = first.toLowerCase();
      second = second.toLowerCase();
    }

    // If the first value should come before the second value...
    if (first < second) {
      // Return -1 for ascending order
      // or 1 for descending order.
      return sortOrder.value === "ascending" ? -1 : 1;
    }

    // If the first value should come after the second value...
    if (first > second) {
      // Return 1 for ascending order
      // or -1 for descending order.
      return sortOrder.value === "ascending" ? 1 : -1;
    }

    // If both values are equal, keep their order unchanged.
    return 0;
  });

  // Return the sorted array.
  return result;
});

// Groups identical lessons in the cart
// and calculates the quantity of each lesson.
const cartItems = computed(() => {
  const groupedItems = [];

  cart.value.forEach((lesson) => {
    const existingItem = groupedItems.find((item) => item.id === lesson.id);

    if (existingItem) {
      existingItem.quantity++;
    } else {
      groupedItems.push({
        ...lesson,
        quantity: 1,
      });
    }
  });

  return groupedItems;
});

// Calculates the total price of all items in the cart.
const cartTotal = computed(() => {
  return cart.value.reduce((total, lesson) => {
    return total + lesson.price;
  }, 0);
});

// Checks if the name and phone number are valid
const isCheckoutValid = computed(() => {
  if (customerName.value === "" || customerPhone.value === "") {
    return false;
  }

  return nameIsValid.value && phoneIsValid.value;
});

//                                        Add to cart function
function addToCart(lesson) {
  if (lesson.spaces > 0) {
    cart.value.push(lesson);
    lesson.spaces--;
  }
}

// Removes one space of a lesson from the cart.
function removeFromCart(lessonId) {
  // Find one matching lesson inside the cart.
  const itemIndex = cart.value.findIndex((item) => item.id === lessonId);

  // Continue only if the lesson exists in the cart.
  if (itemIndex !== -1) {
    // Find the original lesson.
    const originalLesson = lessons.value.find(
      (lesson) => lesson.id === lessonId,
    );

    // Return one available space to the lesson.
    if (originalLesson) {
      originalLesson.spaces++;
    }

    // Remove one item from the cart.
    cart.value.splice(itemIndex, 1);
  }
}
function toggleCart() {
  showCart.value = !showCart.value;
}
// Placing Order
function placeOrder() {
  if (!isCheckoutValid.value) {
    return;
  }

  orderPlaced.value = true;

  cart.value = [];

  customerName.value = "";
  customerPhone.value = "";
}
</script>

<template>
  <div>
    <!-- Navbar -->
    <header class="navbar">
      <div class="logo">
        <span class="logo-icon"
          ><img src="../public/Images/bell-logo.png" alt="Logo"
        /></span>
        <span>BeyondBell</span>
      </div>

      <nav class="nav-links">
        <a href="#">Home</a>
        <a href="#">Classes</a>
        <a href="#">Activities</a>
        <a href="#">Teachers</a>
        <a href="#">About</a>
      </nav>

      <div class="nav-actions">
        <button
          class="cart-button"
          :disabled="cart.length === 0"
          @click="toggleCart"
        >
          🛒

          <span class="cart-count">
            {{ cart.length }}
          </span>
        </button>

        <button class="sign-in-button">Sign In</button>

        <button class="register-button">Register</button>
      </div>
    </header>

    <!-- Hero -->
    <main>
      <template v-if="!showCart">
        <section class="hero">
          <div class="hero-content">
            <p class="hero-label">AFTER SCHOOL CLASSES & ACTIVITIES</p>

            <h1>
              Ignite
              <span class="yellow-text">Curiosity</span>
              <span class="blue-text">Beyond School.</span>
            </h1>

            <p class="hero-description">
              Engaging classes and activities to help every child explore their
              interests, build skills and make new friends.
            </p>

            <!-- Search -->
            <div class="search-box">
              <input
                type="text"
                placeholder="Search for classes, activities or locations..."
              />

              <button>Search</button>
            </div>

            <!-- Filters -->
            <div class="hero-filters">
              <button class="filter active">▦ All</button>

              <button class="filter">🎓 Classes</button>

              <button class="filter">🏃 Activities</button>
            </div>
          </div>

          <!-- Hero visual -->
          <div class="hero-visual">
            <div class="shape shape-one"></div>
            <div class="shape shape-two"></div>

            <img
              class="hero-kids-image"
              src="/Images/hero-kids.png"
              alt="Happy children holding school books"
            />

            <p class="hero-note">
              More than<br />
              just learning
            </p>
          </div>
        </section>

        <!-- Categories -->
        <section class="categories-section">
          <div class="section-title">
            <p>EXPLORE</p>
            <h2>Something for every interest</h2>
            <span>
              Discover after-school activities designed to inspire, challenge
              and entertain.
            </span>
          </div>

          <div class="categories-grid">
            <article
              v-for="category in categories"
              :key="category.name"
              class="category-card"
            >
              <div class="category-icon">
                {{ category.icon }}
              </div>

              <h3>{{ category.name }}</h3>

              <p>
                {{ category.description }}
              </p>
            </article>
          </div>
        </section>

        <!-- Classes & Activities -->
        <section class="lessons-section">
          <div class="lessons-header">
            <div>
              <p class="small-heading">DISCOVER</p>

              <h2>Popular Classes & Activities</h2>

              <p class="lesson-count">
                {{ sortedLessons.length }} experiences available
              </p>
            </div>

            <!-- Sorting -->
            <div class="sorting">
              <label>
                Sort by

                <select v-model="sortBy">
                  <option value="subject">Subject</option>

                  <option value="location">Location</option>

                  <option value="price">Price</option>

                  <option value="spaces">Spaces</option>
                </select>
              </label>

              <label>
                Order

                <select v-model="sortOrder">
                  <option value="ascending">Ascending</option>

                  <option value="descending">Descending</option>
                </select>
              </label>
            </div>
          </div>

          <!-- Lesson Cards -->
          <div class="lesson-grid">
            <article
              v-for="lesson in sortedLessons"
              :key="lesson.id"
              class="lesson-card"
            >
              <div class="lesson-image">
                <img :src="lesson.image" :alt="lesson.subject" />

                <span class="lesson-type" :class="lesson.type.toLowerCase()">
                  {{ lesson.type }}
                </span>
              </div>

              <div class="lesson-content">
                <div class="lesson-title">
                  <h3>
                    {{ lesson.subject }}
                  </h3>

                  <strong> £{{ lesson.price }} </strong>
                </div>

                <p class="lesson-info">📍 {{ lesson.location }}</p>

                <p class="lesson-info">👤 {{ lesson.teacher }}</p>

                <div class="lesson-bottom">
                  <span class="spaces">
                    {{
                      lesson.spaces === 0
                        ? "Fully booked"
                        : lesson.spaces + " spaces left"
                    }}
                  </span>

                  <button
                    @click="addToCart(lesson)"
                    :disabled="lesson.spaces === 0"
                  >
                    {{ lesson.spaces === 0 ? "Sold Out" : "🛒 Add to Cart" }}
                  </button>
                </div>
              </div>
            </article>
          </div>
        </section>
      </template>
      <!-- Cart View -->
      <section v-if="showCart" class="cart-section">
        <div v-if="!orderPlaced">
          <div class="cart-header">
            <div>
              <p class="small-heading">YOUR BASKET</p>

              <h2>Shopping Cart</h2>

              <p>{{ cart.length }} spaces selected</p>
            </div>

            <button class="back-button" @click="showCart = false">
              ← Continue Shopping
            </button>
          </div>

          <div class="cart-layout">
            <!-- Cart Items -->
            <div class="cart-items">
              <template v-if="cartItems.length > 0">
                <article
                  v-for="item in cartItems"
                  :key="item.id"
                  class="cart-item"
                >
                  <img
                    :src="item.image"
                    :alt="item.subject"
                    class="cart-item-image"
                  />

                  <div class="cart-item-info">
                    <span class="cart-item-type">
                      {{ item.type }}
                    </span>

                    <h3>
                      {{ item.subject }}
                    </h3>

                    <p>📍 {{ item.location }}</p>

                    <p>👤 {{ item.teacher }}</p>
                  </div>

                  <div class="cart-item-details">
                    <p>
                      Quantity:
                      <strong>{{ item.quantity }}</strong>
                    </p>

                    <p class="cart-price">£{{ item.price * item.quantity }}</p>

                    <button
                      class="remove-button"
                      @click="removeFromCart(item.id)"
                    >
                      Remove One
                    </button>
                  </div>
                </article>
              </template>
              <!-- Empty basket message -->
              <div v-else class="empty-cart">
                <div class="empty-cart-icon">🛒</div>

                <h3>Your basket is empty</h3>

                <p>Add a class or activity to get started.</p>

                <button @click="showCart = false">
                  Browse Classes & Activities
                </button>
              </div>
            </div>

            <!-- Order Summary -->
            <aside class="cart-summary">
              <h3>Order Summary</h3>

              <div class="summary-row">
                <span> Spaces </span>

                <span>
                  {{ cart.length }}
                </span>
              </div>

              <div class="summary-row total-row">
                <strong>Total</strong>
                <strong>£{{ cartTotal }}</strong>
              </div>

              <!-- Checkout Details -->
              <div class="cart-checkout">
                <h3>Checkout</h3>

                <label for="customer-name"> Full Name </label>

                <input
                  id="customer-name"
                  v-model.trim="customerName"
                  type="text"
                  placeholder="Enter your full name"
                />

                <p
                  v-if="customerName && !nameIsValid"
                  class="validation-message"
                >
                  Enter a name using letters and spaces only.
                </p>

                <label for="customer-phone"> Phone Number </label>

                <input
                  id="customer-phone"
                  v-model="customerPhone"
                  type="tel"
                  placeholder="Enter your phone number"
                />

                <p
                  v-if="customerPhone && !phoneIsValid"
                  class="validation-message"
                >
                  Enter a phone number using digits only.
                </p>

                <button
                  class="place-order-button"
                  :disabled="!isCheckoutValid"
                  @click="placeOrder"
                >
                  Checkout
                </button>
              </div>
            </aside>
          </div>
        </div>

        <!-- Order Confirmation -->
        <div v-else class="order-success">
          <div class="success-icon">✓</div>

          <h2>Order Confirmed</h2>

          <p>Your order has been submitted successfully.</p>

          <button
            class="back-button"
            @click="
              orderPlaced = false;
              showCart = false;
            "
          >
            Back to Classes
          </button>
        </div>
      </section>

      <!-- Checkout View -->
    </main>
  </div>
</template>
