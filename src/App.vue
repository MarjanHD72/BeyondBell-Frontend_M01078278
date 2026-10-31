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

// search
const searchText = ref("");
async function searchLessons() {
  try {
    if (searchText.value.trim() === "") {
      const response = await fetch("http://localhost:3000/lessons");
      const data = await response.json();

      lessons.value = data.map((lesson) => ({
        ...lesson,
        id: lesson._id,
        subject: lesson.topic,
        spaces: lesson.space,
      }));

      return;
    }

    const response = await fetch(
      `http://localhost:3000/lessons/search?q=${encodeURIComponent(searchText.value)}`,
    );

    const data = await response.json();

    lessons.value = data.map((lesson) => ({
      ...lesson,
      id: lesson._id,
      subject: lesson.topic,
      spaces: lesson.space,
    }));
  } catch (error) {
    console.error("Failed to search lessons:", error);
  }
}
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
  fetch("http://localhost:3000/teachers")
    .then((response) => response.json())
    .then((data) => {
      teachers.value = data;
    })
    .catch((error) => {
      console.error("Failed to fetch teachers:", error);
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

//teachers
const teachers = ref([]);
const selectedTeacher = ref("");
// Stores the lessons for the selected teacher
const selectedTeacherLessons = ref([]);

// Stores the full selected teacher details
const selectedTeacherDetails = ref(null);

// Stores the selected teacher name
const selectedTeacherName = ref("");

// Calculates the total price of all items in the cart.
const cartTotal = computed(() => {
  return cart.value.reduce((total, lesson) => {
    return total + lesson.price;
  }, 0);
});
//Filtering Teachers List
async function filterByTeacher() {
  try {
    if (selectedTeacher.value === "") {
      const response = await fetch("http://localhost:3000/lessons");
      const data = await response.json();
      lessons.value = data.map((lesson) => ({
        ...lesson,
        id: lesson._id,
        subject: lesson.topic,
        spaces: lesson.space,
      }));

      return;
    }

    const response = await fetch(
      `http://localhost:3000/teachers/${selectedTeacher.value}/lessons`,
    );

    const data = await response.json();

    lessons.value = data.map((lesson) => ({
      ...lesson,
      id: lesson._id,
      subject: lesson.topic,
      spaces: lesson.space,
    }));
  } catch (error) {
    console.error("Failed to filter lessons by teacher:", error);
  }
}
// Gets the lessons for the selected teacher
async function showTeacherLessons(teacher) {
  selectedTeacher.value = teacher._id;

  selectedTeacherDetails.value = teacher;

  selectedTeacherName.value = teacher.firstName + " " + teacher.lastName;

  try {
    const response = await fetch(
      `http://localhost:3000/teachers/${teacher._id}/lessons`,
    );

    const data = await response.json();

    selectedTeacherLessons.value = data.map((lesson) => {
      const currentLesson = lessons.value.find(
        (item) => item.id === lesson._id,
      );

      return {
        ...lesson,
        id: lesson._id,
        subject: lesson.topic,
        spaces: currentLesson ? currentLesson.spaces : lesson.space,
      };
    });
  } catch (error) {
    console.error("Failed to get teacher lessons:", error);
  }
}
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

    // Update lesson in main lessons list
    const mainLesson = lessons.value.find((item) => item.id === lesson.id);

    if (mainLesson) {
      mainLesson.spaces--;
    }

    // Update lesson inside teacher modal too
    const teacherLesson = selectedTeacherLessons.value.find(
      (item) => item.id === lesson.id,
    );

    if (teacherLesson && teacherLesson !== mainLesson) {
      teacherLesson.spaces--;
    }
  }
}

// Removes one space of a lesson from the cart.
function removeFromCart(lessonId) {
  const itemIndex = cart.value.findIndex((item) => item.id === lessonId);

  if (itemIndex !== -1) {
    const mainLesson = lessons.value.find((lesson) => lesson.id === lessonId);

    if (mainLesson) {
      mainLesson.spaces++;
    }

    const teacherLesson = selectedTeacherLessons.value.find(
      (lesson) => lesson.id === lessonId,
    );

    if (teacherLesson && teacherLesson !== mainLesson) {
      teacherLesson.spaces++;
    }

    cart.value.splice(itemIndex, 1);
  }
}
function toggleCart() {
  showCart.value = !showCart.value;
}
// Placing Order
async function placeOrder() {
  if (!isCheckoutValid.value) {
    return;
  }

  const order = {
    name: customerName.value,
    phone: customerPhone.value,

    lessonIDs: cartItems.value.map((item) => item.id),

    items: cartItems.value.map((item) => ({
      lessonId: item.id,
      spaces: item.quantity,
    })),

    spaces: cart.value.length,
  };

  try {
    // Create the order
    const response = await fetch("http://localhost:3000/orders", {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
      },
      body: JSON.stringify(order),
    });

    const data = await response.json();

    console.log(data);

    // Get each lesson only once
    const lessonsToUpdate = [
      ...new Map(cart.value.map((lesson) => [lesson.id, lesson])).values(),
    ];

    // Update remaining spaces in MongoDB
    for (const lesson of lessonsToUpdate) {
      await fetch(`http://localhost:3000/lessons/${lesson.id}`, {
        method: "PUT",
        headers: {
          "Content-Type": "application/json",
        },
        body: JSON.stringify({
          space: lesson.spaces,
        }),
      });
    }

    orderPlaced.value = true;

    cart.value = [];

    customerName.value = "";
    customerPhone.value = "";
  } catch (error) {
    console.error("Failed to place order:", error);
  }
}
// Stores the current page
const currentView = ref("home");
// Filters only class lessons
const classLessons = computed(() => {
  return lessons.value.filter((lesson) => lesson.type === "Class");
});
// Filters only activity lessons
const activityLessons = computed(() => {
  return lessons.value.filter((lesson) => lesson.type === "Activity");
});
// hamburger Menu
const menuOpen = ref(false);
// Change page, close cart and mobile menu
function changeView(view) {
  currentView.value = view;
  showCart.value = false;
  menuOpen.value = false;
}

// Register form
const registerName = ref("");
const registerEmail = ref("");
const registerPassword = ref("");
const registerConfirmPassword = ref("");

const registerMessage = ref("");

// Checks if register form is valid
const registerIsValid = computed(() => {
  return (
    registerName.value.trim() !== "" &&
    registerEmail.value.trim() !== "" &&
    registerPassword.value !== "" &&
    registerConfirmPassword.value !== "" &&
    registerPassword.value === registerConfirmPassword.value
  );
});

// Register User
async function registerUser() {
  registerMessage.value = "";

  if (!registerIsValid.value) {
    registerMessage.value = "Please complete the form correctly.";
    return;
  }

  try {
    const response = await fetch("http://localhost:3000/register", {
      method: "POST",

      headers: {
        "Content-Type": "application/json",
      },

      body: JSON.stringify({
        name: registerName.value,
        email: registerEmail.value,
        password: registerPassword.value,
      }),
    });

    const data = await response.json();

    registerMessage.value = data.message;

    if (response.ok) {
      registerName.value = "";
      registerEmail.value = "";
      registerPassword.value = "";
      registerConfirmPassword.value = "";
    }
  } catch (error) {
    console.error("Registration failed:", error);

    registerMessage.value = error.message;
  }
}

// Login form data
const loginEmail = ref("");
const loginPassword = ref("");
const loginMessage = ref("");
// Store the logged in user
const loggedInUser = ref(null);

// Login V alidation_ Check if the login form is complete
const loginIsValid = computed(() => {
  return loginEmail.value.trim() !== "" && loginPassword.value !== "";
});
// Login user
async function loginUser() {
  loginMessage.value = "";

  if (!loginIsValid.value) {
    loginMessage.value = "Please enter your email and password.";
    return;
  }

  try {
    const response = await fetch("http://localhost:3000/login", {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
      },
      body: JSON.stringify({
        email: loginEmail.value,
        password: loginPassword.value,
      }),
    });

    const data = await response.json();

    loginMessage.value = data.message;

    if (response.ok) {
      loggedInUser.value = data.user;
      console.log("Logged in user:", data.user);
      loginEmail.value = "";
      loginPassword.value = "";
      // Go to the home page after successful login
      currentView.value = "home";
      // Scroll to the top after login
      window.scrollTo({
        top: 0,
        behavior: "smooth",
      });
    }
  } catch (error) {
    console.error("Login failed:", error);
    loginMessage.value = "Login failed.";
  }
}
// Logout user
function logoutUser() {
  console.log("Logout clicked");

  loggedInUser.value = null;
  loginMessage.value = "";
  currentView.value = "home";
  showCart.value = false;
  menuOpen.value = false;
}
</script>

<!-- Main Template  -->
<template>
  <div>
    <!-- Top header -->
    <header class="navbar">
      <div class="logo">
        <span class="logo-icon"
          ><img src="../public/Images/bell-logo.png" alt="Logo"
        /></span>
        <span>BeyondBell</span>
      </div>

      <button
        class="nav-toggle"
        @click="menuOpen = !menuOpen"
        aria-label="Toggle menu"
      >
        <span></span>
        <span></span>
        <span></span>
      </button>
      <nav class="nav-links" :class="{ 'is-open': menuOpen }">
        <a
          href="#"
          :class="{ active: currentView === 'home' }"
          @click.prevent="changeView('home')"
        >
          Home
        </a>

        <a
          href="#"
          :class="{ active: currentView === 'classes' }"
          @click.prevent="changeView('classes')"
        >
          Classes
        </a>

        <a
          href="#"
          :class="{ active: currentView === 'activities' }"
          @click.prevent="changeView('activities')"
        >
          Activities
        </a>

        <a
          href="#"
          :class="{ active: currentView === 'teachers' }"
          @click.prevent="changeView('teachers')"
        >
          Teachers
        </a>

        <a
          href="#"
          :class="{ active: currentView === 'about' }"
          @click.prevent="changeView('about')"
        >
          About
        </a>

        <div class="mobile-menu-actions">
          <button class="sign-in-button" type="button">Sign In</button>
          <button class="register-button" @click="changeView('register')">
            Register
          </button>
        </div>
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

        <div class="auth-buttons">
          <template v-if="loggedInUser">
            <span class="user-name"> Hello, {{ loggedInUser.name }} </span>

            <button type="button" class="login-button" @click="logoutUser">
              Logout
            </button>
          </template>

          <template v-else>
            <button class="login-button" @click="changeView('login')">
              Login
            </button>

            <button class="register-button" @click="changeView('register')">
              Register
            </button>
          </template>
        </div>
      </div>
    </header>

    <!-- Main page area -->
    <main>
      <template v-if="!showCart">
        <template v-if="currentView === 'home'">
          <section class="hero">
            <div class="hero-content">
              <p class="hero-label">AFTER SCHOOL CLASSES & ACTIVITIES</p>

              <h1>
                Ignite
                <span class="yellow-text">Curiosity</span>
                <span class="blue-text">Beyond School.</span>
              </h1>

              <p class="hero-description">
                Engaging classes and activities to help every child explore
                their interests, build skills and make new friends.
              </p>

              <!-- Search -->
              <div class="search-box">
                <input
                  v-model="searchText"
                  type="text"
                  placeholder="Search for classes, activities or locations..."
                  @keyup.enter="searchLessons"
                />

                <button @click="searchLessons">Search</button>
              </div>

              <!-- Filters -->
              <div class="hero-filters">
                <button
                  class="filter"
                  :class="{ active: currentView === 'home' }"
                  @click="currentView = 'home'"
                >
                  ▦ All
                </button>

                <button
                  class="filter"
                  :class="{ active: currentView === 'classes' }"
                  @click="currentView = 'classes'"
                >
                  🎓 Classes
                </button>

                <button
                  class="filter"
                  :class="{ active: currentView === 'activities' }"
                  @click="currentView = 'activities'"
                >
                  🏃 Activities
                </button>
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
            </div>
          </section>
          <div
            class="home-sections"
            :class="{ 'search-active': searchText.trim() !== '' }"
          >
            <!-- Categories -->
            <section class="categories-section">
              <div class="section-title">
                <p>EXPLORE</p>
                <h2>Something for every interest</h2>
                <span>
                  Discover after-school activities designed to inspire,
                  challenge and entertain.
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
                  <div class="teacher-filter">
                    <label>
                      Teacher

                      <select
                        v-model="selectedTeacher"
                        @change="filterByTeacher"
                      >
                        <option value="">All Teachers</option>

                        <option
                          v-for="teacher in teachers"
                          :key="teacher._id"
                          :value="teacher._id"
                        >
                          {{ teacher.firstName }} {{ teacher.lastName }}
                        </option>
                      </select>
                    </label>
                  </div>
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

                    <span
                      class="lesson-type"
                      :class="lesson.type.toLowerCase()"
                    >
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
                        {{
                          lesson.spaces === 0 ? "Sold Out" : "🛒 Add to Cart"
                        }}
                      </button>
                    </div>
                  </div>
                </article>
              </div>
              <div v-if="lessons.length === 0" class="no-results">
                <div class="no-results-icon">🔍</div>

                <h3>No lessons found</h3>

                <p>
                  Try searching for a different class, activity or location.
                </p>
              </div>
            </section>
          </div>
        </template>
        <!-- Classes page -->
        <template v-if="currentView === 'classes'">
          <section class="lessons-section page-lessons-section">
            <div class="lessons-header">
              <div>
                <p class="small-heading">DISCOVER</p>
                <h2>Classes</h2>
                <p class="lesson-count">
                  {{ classLessons.length }} classes available
                </p>
              </div>
            </div>

            <div class="lesson-grid">
              <article
                v-for="lesson in classLessons"
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
                    <h3>{{ lesson.subject }}</h3>
                    <strong>£{{ lesson.price }}</strong>
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

        <!-- Activity page -->
        <template v-if="currentView === 'activities'">
          <section class="lessons-section page-lessons-section">
            <div class="lessons-header">
              <div>
                <p class="small-heading">DISCOVER</p>
                <h2>Activities</h2>
                <p class="lesson-count">
                  {{ activityLessons.length }} activities available
                </p>
              </div>
            </div>

            <div class="lesson-grid">
              <article
                v-for="lesson in activityLessons"
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
                    <h3>{{ lesson.subject }}</h3>
                    <strong>£{{ lesson.price }}</strong>
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
        <!-- teachers page -->
        <template v-if="currentView === 'teachers'">
          <section class="lessons-section">
            <div class="lessons-header">
              <div>
                <p class="small-heading">OUR TEAM</p>
                <h2>Meet Our Teachers</h2>
                <p class="lesson-count">
                  Meet our teachers and explore their lessons.
                </p>
              </div>
            </div>

            <div class="lesson-grid">
              <article
                v-for="teacher in teachers"
                :key="teacher._id"
                class="lesson-card"
              >
                <div class="lesson-content">
                  <img
                    :src="teacher.image"
                    :alt="teacher.firstName + ' ' + teacher.lastName"
                    class="teacher-avatar"
                  />

                  <div class="lesson-title">
                    <h3>
                      {{ teacher.firstName }}
                      {{ teacher.lastName }}
                    </h3>
                  </div>

                  <p class="lesson-info">
                    {{ teacher.lessonTopics.join(", ") }}
                  </p>

                  <div class="lesson-bottom">
                    <button @click="showTeacherLessons(teacher)">
                      View Teacher
                    </button>
                  </div>
                </div>
              </article>
            </div>
          </section>
        </template>
        <!-- Teacher Modal -->
        <Transition name="teacher-modal">
          <div
            v-if="selectedTeacherDetails"
            class="teacher-modal-overlay"
            @click.self="selectedTeacherDetails = null"
          >
            <div class="teacher-modal">
              <button
                class="teacher-modal-close"
                @click="selectedTeacherDetails = null"
              >
                ×
              </button>

              <div class="teacher-modal-header">
                <img
                  :src="selectedTeacherDetails.image"
                  :alt="
                    selectedTeacherDetails.firstName +
                    ' ' +
                    selectedTeacherDetails.lastName
                  "
                  class="teacher-avatar"
                />

                <div>
                  <p class="small-heading">TEACHER PROFILE</p>

                  <h2>
                    {{ selectedTeacherDetails.firstName }}
                    {{ selectedTeacherDetails.lastName }}
                  </h2>

                  <p>
                    {{ selectedTeacherDetails.lessonTopics.join(", ") }}
                  </p>
                </div>
              </div>

              <div class="teacher-modal-lessons">
                <p class="small-heading">AVAILABLE LESSON</p>

                <div class="lesson-grid">
                  <article
                    v-for="lesson in selectedTeacherLessons"
                    :key="lesson.id"
                    class="lesson-card"
                  >
                    <div class="lesson-image">
                      <img :src="lesson.image" :alt="lesson.subject" />

                      <span
                        class="lesson-type"
                        :class="lesson.type.toLowerCase()"
                      >
                        {{ lesson.type }}
                      </span>
                    </div>

                    <div class="lesson-content">
                      <div class="lesson-title">
                        <h3>{{ lesson.subject }}</h3>

                        <strong>£{{ lesson.price }}</strong>
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
                          {{
                            lesson.spaces === 0 ? "Sold Out" : "🛒 Add to Cart"
                          }}
                        </button>
                      </div>
                    </div>
                  </article>
                </div>
              </div>
            </div>
          </div></Transition
        >
      </template>
      <!-- About page -->
      <template v-if="currentView === 'about'">
        <section class="about-section">
          <div class="about-content">
            <p class="small-heading">ABOUT BEYONDBELL</p>

            <h2>Learning Beyond the Classroom</h2>

            <p>
              BeyondBell helps parents discover engaging after-school classes
              and activities for children.
            </p>

            <p>
              From coding and science to music, sports and creative arts,
              children can explore their interests, develop new skills and enjoy
              learning in a fun and supportive environment.
            </p>

            <p>
              Our aim is to make it easier for families to find suitable
              activities, view available spaces and book lessons in one place.
            </p>
          </div>
        </section>
      </template>
      <!-- Register page -->
      <template v-if="currentView === 'register'">
        <section class="register-section">
          <div class="register-card">
            <p class="small-heading">CREATE ACCOUNT</p>

            <h2>Register</h2>

            <p class="register-text">
              Create your BeyondBell account to book classes and activities.
            </p>

            <form class="register-form" @submit.prevent="registerUser">
              <label>
                Full Name

                <input
                  v-model.trim="registerName"
                  type="text"
                  placeholder="Enter your full name"
                />
              </label>

              <label>
                Email

                <input
                  v-model.trim="registerEmail"
                  type="email"
                  placeholder="Enter your email"
                />
              </label>

              <label>
                Password

                <input
                  v-model="registerPassword"
                  type="password"
                  placeholder="Enter your password"
                />
              </label>

              <label>
                Confirm Password

                <input
                  v-model="registerConfirmPassword"
                  type="password"
                  placeholder="Confirm your password"
                />
              </label>

              <p
                v-if="
                  registerConfirmPassword &&
                  registerPassword !== registerConfirmPassword
                "
                class="validation-message"
              >
                Passwords do not match.
              </p>

              <p v-if="registerMessage" class="register-message">
                {{ registerMessage }}
              </p>

              <button
                type="submit"
                class="register-submit-button"
                :disabled="!registerIsValid"
              >
                Register
              </button>
            </form>
          </div>
        </section>
      </template>
      <!-- Login page -->
      <template v-if="currentView === 'login'">
        <section class="register-section">
          <div class="register-card">
            <p class="small-heading">WELCOME BACK</p>

            <h2>Login</h2>

            <p class="register-text">Login to your BeyondBell account.</p>

            <form class="register-form" @submit.prevent="loginUser">
              <label>
                Email
                <input
                  v-model.trim="loginEmail"
                  type="email"
                  placeholder="Enter your email"
                />
              </label>

              <label>
                Password
                <input
                  v-model="loginPassword"
                  type="password"
                  placeholder="Enter your password"
                />
              </label>

              <p v-if="loginMessage" class="register-message">
                {{ loginMessage }}
              </p>

              <button
                type="submit"
                class="register-submit-button"
                :disabled="!loginIsValid"
              >
                Login
              </button>
            </form>
          </div>
        </section>
      </template>
      <!-- Cart view -->
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
