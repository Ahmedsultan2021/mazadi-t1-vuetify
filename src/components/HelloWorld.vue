<template>
  <v-container>
    <v-row class="text-center">
      <v-col>
        <v-autocomplete
          v-model="selectedMainCategory"
          :items="mainCategories"
          item-text="name"
          item-value="id"
          @change="getSubcategories"
          label="Main Category"
        ></v-autocomplete>

        <v-autocomplete
          v-model="selectedSubcategory"
          :items="subcategories"
          item-text="name"
          item-value="id"
          @change="getProperties"
          label="SubCategory"
        ></v-autocomplete>

        <v-row v-if="properties.length > 0">
          <v-col cols="12" v-for="(property, index) in properties" :key="index">
            <v-autocomplete
              v-model="selectedProperties[index]"
              :items="[...property.options, { id: 'other', name: 'Other' }]"
              item-text="name"
              item-value="id"
              @change="fetchChildOptions(property, index)"
              clearable
              :label="property.name"
            >
            </v-autocomplete>
            <v-text-field
              v-if="selectedProperties[index] === 'other'"
              v-model="otherValues[index]"
            ></v-text-field>
          </v-col>
        </v-row>
        <v-row v-else>
          <v-col>
            <!-- Display a message or alternative content when properties is empty -->
            <p>No properties available.</p>
          </v-col>
        </v-row>

        <v-btn @click="clearForm" style="margin-top: 30px">clear Form</v-btn>
        <v-btn @click="displayForm" style="margin-top: 30px"
          >Display Form Content</v-btn
        >
      </v-col>
      <v-col>
        <v-simple-table v-if="displayTable">
          <template v-slot:default>
            <thead>
              <tr>
                <th class="text-left">Name</th>
                <th class="text-left">Calories</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="(data, index) in tableData" :key="index">
                <td>{{ data.Property }}</td>
                <td>{{ data.Value }}</td>
              </tr>
            </tbody>
          </template>
        </v-simple-table>
      </v-col>
    </v-row>
  </v-container>
</template>
<script>
import axios from "../../axios.config";

const API_URL_ALL_CATEGORIES =
  "https://staging.mazaady.com/api/v1/get_all_cats";
const API_URL_PROPERTIES = "https://staging.mazaady.com/api/v1/properties?cat=";
const API_URL_OPTIONS_CHILD =
  "https://staging.mazaady.com/api/v1/get-options-child/";

export default {
  data() {
    return {
      privateApiKey: "3%o8i}_;3D4bF]G5@22r2)Et1&mLJ4?$@+16",
      selectedMainCategory: null,
      selectedSubcategory: null,
      selectedchildprop: null,
      selectedProperties: [],
      otherValues: [],
      selectedOption: null,
      mainCategories: [],
      subcategories: [],
      properties: [],
      label1: null,
      label2: null,
      options: [],
      displayTable: false,
      tableHeaders: ["Property", "Value"],
      tableData: [],
      customOptionValue: null,
      childPropperities: [],
      allData: [],
      currentIndex: 1, // Index tracker for current element\
      propsArray: [],
    };
  },
  watch: {
    selectedMainCategory(newVal) {
      this.properties = [];
      this.tableData = [];
      this.currentIndex = 1;
      if (newVal === null) {
        console.log("selectedMainCategory is null!");
      }
    },
    selectedSubcategory(newVal) {
      this.properties = [];
      this.tableData = [];
      this.currentIndex = 1;
      if (newVal === null) {
        console.log("selectedMainCategory is null!");
      }
    },
   

  },
  mounted() {
    this.getMainCategories();
  },

  methods: {
    async getMainCategories() {
      this.subcategories = [];
      this.properties = [];
      this.options = [];

      try {
        const response = await axios.get(API_URL_ALL_CATEGORIES, {
          headers: {
            "private-key": this.privateApiKey,
          },
        });
        this.mainCategories = response.data.data.categories;
      } catch (error) {
        console.error(error);
      }
    },

    addnextprop() {
      this.displayFormContent();
      if (this.currentIndex < this.allData.length) {
        const nextProperty = this.allData[this.currentIndex];
        console.log("xxxxxxxxxxxxxxxxxx");
        this.propsArray = this.properties.map((obj) => obj.name).flat();
        console.log(this.propsArray);
        console.log(nextProperty);
        console.log("xxxxxxxxxxxxxxxxxx");
        if (!this.propsArray.includes(nextProperty.name)) {
          this.properties.push(nextProperty);
          this.currentIndex++;
        }
      }
    },
    async getSubcategories() {
      const selectedCategory = this.mainCategories.find(
        (category) => category.id === this.selectedMainCategory
      );
      this.subcategories = selectedCategory?.children || [];
      this.displayTable = false;
    },
    async getProperties() {
      this.selectedProperties = [];
      this.displayTable = false;
      this.otherValues = [];
      this.options = [];
      this.selectedOption = "";
      try {
        const apiUrl = `${API_URL_PROPERTIES}${this.selectedSubcategory}`;
        const response = await axios.get(apiUrl, {
          headers: {
            "private-key": this.privateApiKey,
          },
        });
        this.allData = response.data.data;
        console.log(this.allData);
        const firstProperty = response.data.data[0];
        this.properties.push(firstProperty);
      } catch (error) {
        console.error(error);
      }
    },
    async fetchChildOptions(property, index) {
      this.displayFormContent();
      this.displayTable = false;
      const selectedProperty = this.selectedProperties[index];
      if (selectedProperty !== "other") {
        try {
          const apiUrl = `${API_URL_OPTIONS_CHILD}${selectedProperty}`;
          const response = await axios.get(apiUrl, {
            headers: {
              "private-key": this.privateApiKey,
            },
          });
          this.label1 = response.data.data[0].name;
          this.options = response.data.data[0]?.options || [];
          const propertie = response.data.data[0];
          this.propsArray = this.properties.map((obj) => obj.name).flat();
          console.log(this.propsArray);
          console.log(propertie);

          if (!this.propsArray.includes(propertie.name)) {
            this.properties.push(propertie);
          }
        } catch (error) {
          console.error(error);
          this.addnextprop();
        }
      } else {
        this.addnextprop();
      }
    },

    closeForm() {
      this.displayTable = false;
    },
    displayFormContent() {
      this.tableData = []; // Clear the table data
      if (this.selectedMainCategory) {
        const selectedMainCategory = this.mainCategories.find(
          (category) => category.id === this.selectedMainCategory
        );
        this.tableData.push({
          Property: "Main Category",
          Value: selectedMainCategory.name,
        });
      }
      console.log(this.tableData);

      if (this.selectedSubcategory) {
        const selectedSubcategory = this.subcategories.find(
          (subcategory) => subcategory.id === this.selectedSubcategory
        );
        this.tableData.push({
          Property: "Subcategory",
          Value: selectedSubcategory.name,
        });
      }

      this.properties.forEach((property, index) => {
        const selectedValue = this.selectedProperties[index];
        let value = "";

        if (selectedValue !== "other") {
          const selectedOption = property.options.find(
            (option) => option.id === selectedValue
          );
          if (selectedOption) {
            value = selectedOption.name;
          }
        } else {
          value = this.otherValues[index];
        }

        if (value !== "") {
          this.tableData.push({
            Property: property.name,
            Value: value,
          });
        }
      });

      if (this.selectedOption) {
        this.tableData.push({
          Property: "Model",
          Value: this.customOptionValue
            ? this.customOptionValue
            : this.selectedOption.name,
        });
      }
    },
    displayForm() {
      this.displayFormContent();
      this.displayTable = true; // Show the table
    },
    clearForm() {
      this.displayFormContent();
      this.tableData = []; // Show the table
      this.properties = []; // Show the table
      this.selectedProperties = [];
      this.selectedMainCategory = null;
      this.selectedSubcategory = null;
      this.currentIndex = 1;
      this.getMainCategories()
        .then(() => {
          return this.getSubcategories();
        })

        .catch((error) => {
          console.log(error);
        });
    },
  },
};
</script>
