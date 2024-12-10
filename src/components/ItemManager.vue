<template>
    <div class="item-manager">
      <!-- Search Bar with Autocomplete -->
      <header class="search-bar">
        <input
          type="text"
          v-model="itemCode"
          placeholder="Enter Item Code"
          @input="fetchItemSuggestions"
        />
        <ul v-if="suggestions.length" class="suggestions">
          <li
            v-for="(suggestion, index) in suggestions"
            :key="index"
            @click="selectSuggestion(suggestion.code)"
          >
            {{ suggestion.name }}
          </li>
        </ul>
        <!-- Single Button: Toggles between fetching and updating -->
        <button @click="handleAction">{{ buttonText }}</button>
      </header>
  
      <!-- Error Message -->
      <div v-if="error" class="error">{{ error }}</div>
  
      <!-- Modify Item Form (Only shown if item is fetched) -->
      <div v-if="item">
        <h2>Modify Item Details</h2>
        <form @submit.prevent="updateItem">
          <!-- Image Section -->
          <div class="image-section">
            <h3>Item Images</h3>
  
            <!-- Front Image -->
            <div class="image-field">
              <label for="custom_img_front">Front Image:</label>
              <div v-if="item.custom_img_front">
                <img :src="item.custom_img_front" alt="Front Image" width="100" height="100" />
              </div>
              <input
                id="custom_img_front"
                type="file"
                @change="handleImageUpload('custom_img_front', $event)"
                accept="image/*"
              />
            </div>
  
            <!-- Rear Image -->
            <div class="image-field">
              <label for="custom_img_rear">Rear Image:</label>
              <div v-if="item.custom_img_rear">
                <img :src="item.custom_img_rear" alt="Rear Image" width="100" height="100" />
              </div>
              <input
                id="custom_img_rear"
                type="file"
                @change="handleImageUpload('custom_img_rear', $event)"
                accept="image/*"
              />
            </div>
  
            <!-- Right Image -->
            <div class="image-field">
              <label for="custom_img_right">Right Image:</label>
              <div v-if="item.custom_img_right">
                <img :src="item.custom_img_right" alt="Right Image" width="100" height="100" />
              </div>
              <input
                id="custom_img_right"
                type="file"
                @change="handleImageUpload('custom_img_right', $event)"
                accept="image/*"
              />
            </div>
  
            <!-- Left Image -->
            <div class="image-field">
              <label for="custom_img_left">Left Image:</label>
              <div v-if="item.custom_img_left">
                <img :src="item.custom_img_left" alt="Left Image" width="100" height="100" />
              </div>
              <input
                id="custom_img_left"
                type="file"
                @change="handleImageUpload('custom_img_left', $event)"
                accept="image/*"
              />
            </div>
          </div>
  
          <!-- Item Name -->
          <div>
            <label for="name">Name:</label>
            <input id="name" v-model="item.name" type="text" />
          </div>
  
          <!-- Brand -->
          <div>
            <label for="brand">Brand:</label>
            <input id="brand" v-model="item.brand" type="text" />
          </div>
  
          <!-- Price -->
          <div>
            <label for="price">Price:</label>
            <input id="price" v-model="item.price" type="number" step="0.01" />
          </div>
  
          <!-- Description -->
          <div>
            <label for="description">Description:</label>
            <textarea id="description" v-model="item.description"></textarea>
          </div>
  
          <!-- Submit Button -->
          <button type="submit">Update Item</button>
        </form>
      </div>
    </div>
  </template>
  
  <script>
  import axios from "axios";
  
  export default {
    data() {
      return {
        itemCode: "", // Item code entered by the user
        item: null, // Fetched item data
        error: null, // Error message
        isFetching: true, // Flag to track if we are fetching or updating
        suggestions: [], // Suggestions for the item name
      };
    },
    computed: {
      // Dynamic button text based on the current action (fetch or update)
      buttonText() {
        return this.isFetching ? "Fetch Item" : "Update Item";
      },
    },
    methods: {
      // Handle action for the button: either fetch item or update item
      async handleAction() {
        if (this.isFetching) {
          this.fetchItem(); // Fetch item details if we're in "fetch" mode
        } else {
          this.updateItem(); // Update item details if we're in "update" mode
        }
      },
  
      // Fetch suggestions based on user input
      async fetchItemSuggestions() {
        try {
          const response = await axios.get(
            `https://yash.tranqwality.com/api/resource/Item?filters=[["name", "like", "%${this.itemCode}%"]]`,
            {
              headers: {
                Authorization: "token b73c5b96959aa53:1d8250a9d955d9b",
              },
            }
          );
          this.suggestions = response.data.data.map((item) => ({
            name: item.name,
            code: item.name,
          }));
        } catch (err) {
          console.error("Error fetching suggestions: ", err);
          this.suggestions = [];
        }
      },
  
      // Select a suggestion
      selectSuggestion(code) {
        this.itemCode = code;
        this.suggestions = [];
        this.fetchItem();
      },
  
      // Fetch Item Details
      async fetchItem() {
        this.error = null; // Clear previous errors
        this.item = null; // Clear previous item data
  
        if (!this.itemCode.trim()) {
          this.error = "Please enter a valid item code.";
          return;
        }
  
        try {
          const response = await axios.get(
            `https://yash.tranqwality.com/api/resource/Item/${this.itemCode}`,
            {
              headers: {
                Authorization: "token b73c5b96959aa53:1d8250a9d955d9b",
              },
            }
          );
          this.item = response.data.data;
  
          // Initialize images if they exist
          this.item.custom_img_front = this.item.custom_img_front || null;
          this.item.custom_img_rear = this.item.custom_img_rear || null;
          this.item.custom_img_right = this.item.custom_img_right || null;
          this.item.custom_img_left = this.item.custom_img_left || null;
  
          this.isFetching = false; // Switch to "update" mode after fetching item
        } catch (err) {
          this.error = "Failed to fetch item. Please check the item code.";
          console.error(err);
        }
      },
  
      // Update Item Details
      async updateItem() {
        this.error = null; // Clear previous errors
  
        // Construct the data to be sent to the API
        const itemData = {
          name: this.item.name, // Item name
          brand: this.item.brand,
          price: this.item.price,
          description: this.item.description, // Description
          custom_img_front: this.item.custom_img_front,
          custom_img_rear: this.item.custom_img_rear,
          custom_img_right: this.item.custom_img_right,
          custom_img_left: this.item.custom_img_left,
        };
  
        try {
          const response = await axios.put(
            `https://yash.tranqwality.com/api/resource/Item/${this.itemCode}`,
            itemData,
            {
              headers: {
                Authorization: "token b73c5b96959aa53:1d8250a9d955d9b",
              },
            }
          );
          alert("Item updated successfully!");
          console.log(response.data);
        } catch (err) {
          console.error("Error updating item: ", err);
          this.error = "Failed to update item. Please try again.";
        }
      },
  
      // Handle Image Upload and Convert to Base64
      handleImageUpload(field, event) {
        const file = event.target.files[0];
        if (file && file.type.startsWith("image/")) {
          const reader = new FileReader();
          reader.onloadend = () => {
            this.item[field] = reader.result; // Store Base64 encoded image in the correct field
          };
          reader.readAsDataURL(file); // Read file as Base64
        } else {
          this.error = "Please upload a valid image file.";
        }
      },
    },
  };
  </script>
  
  <style scoped>
  /* Styling for suggestions dropdown */
  .suggestions {
    list-style: none;
    padding: 0;
    margin: 0;
    border: 1px solid #ccc;
    border-radius: 4px;
    background-color: #fff;
    position: absolute;
    z-index: 1000;
    max-height: 150px;
    overflow-y: auto;
  }
  
  .suggestions li {
    padding: 10px;
    cursor: pointer;
  }
  
  .suggestions li:hover {
    background-color: #007bff;
    color: white;
  }
  
  /* Other styles... */
  /* Styling for the item manager */
  .item-manager {
    max-width: 600px;
    margin: 0 auto;
    padding: 20px;
    font-family: Arial, sans-serif;
    color: #333;
  }
  
  /* Search Bar Styling */
  .search-bar {
    display: flex;
    align-items: center;
    margin-bottom: 20px;
  }
  
  .search-bar input {
    flex: 1;
    padding: 8px;
    margin-right: 10px;
    border: 1px solid #ccc;
    border-radius: 4px;
  }
  
  .search-bar button {
    padding: 8px 12px;
    border: none;
    background-color: #007bff;
    color: white;
    cursor: pointer;
    border-radius: 4px;
  }
  
  .search-bar button:hover {
    background-color: #0056b3;
  }
  
  /* Error Styling */
  .error {
    color: red;
    margin-top: 10px;
  }
  
  /* Form Styling */
  form {
    background-color: #f9f9f9;
    padding: 15px;
    border: 1px solid #ddd;
    border-radius: 4px;
  }
  
  form div {
    margin-bottom: 10px;
  }
  
  form label {
    display: block;
    font-weight: bold;
    margin-bottom: 5px;
  }
  
  form input,
  form textarea {
    width: 100%;
    padding: 8px;
    border: 1px solid #ccc;
    border-radius: 4px;
  }
  
  form textarea {
    resize: vertical;
  }
  
  form button {
    display: inline-block;
    padding: 10px 15px;
    border: none;
    background-color: #28a745;
    color: white;
    cursor: pointer;
    border-radius: 4px;
  }
  
  form button:hover {
    background-color: #218838;
  }
  
  /* Image Section Styling */
  .image-section {
    margin-bottom: 20px;
  }
  
  .image-field {
    margin-bottom: 15px;
  }
  
  .image-field img {
    margin-top: 10px;
    max-width: 100px;
    height: auto;
  }
  
  .image-field input {
    margin-top: 5px;
  }
  
  .image-field label {
    display: block;
    font-weight: bold;
  }
  </style>
  