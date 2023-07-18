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

        <v-row>
          <v-col
            cols="12"
            v-for="(property, index) in properties.slice(0, 3)"
            :key="index"
          >
            <v-autocomplete
              v-model="selectedProperties[index]"
              :items="[...property.options, { id: 'other', name: 'Other' }]"
              item-text="name"
              item-value="id"
              @change="fetchChildOptions(property, index)"
              clearable
              :label="property.name"
            >
              <template v-slot:selection="{ item }">
                {{ item.name }}
              </template>
              <template v-slot:item="{ item }">
                {{ item.name }}
              </template>
            </v-autocomplete>
            <v-text-field
              v-if="selectedProperties[index] === 'other'"
              v-model="otherValues[index]"
            ></v-text-field>
          </v-col>
        </v-row>

        <div v-if="options.length">
          <v-autocomplete
            v-model="selectedOption"
            :items="[...options, { id: 'other', name: 'Other' }]"
            item-text="name"
            item-value="id"
            label="Model"
            clearable
            return-object
            @change="handleOptionChange"
          ></v-autocomplete>
        </div>

        <v-text-field
          v-if="selectedOption && selectedOption.id === 'other'"
          v-model="customOptionValue"
          label="Other Value"
        ></v-text-field>

        <v-btn @click="displayFormContent" style="margin-top:30px" >Display Form Content</v-btn>
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
      selectedProperties: [],
      otherValues: [],
      selectedOption: null,
      mainCategories: [],
      subcategories: [],
      properties: [],
      options: [],
      displayTable: false,
      tableHeaders: ["Property", "Value"],
      tableData: [],
      customOptionValue:null
    };
  },
  mounted() {
    this.getMainCategories();
  },
  methods: {
    async getMainCategories() {
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
    async getSubcategories() {
      const selectedCategory = this.mainCategories.find(
        (category) => category.id === this.selectedMainCategory
      );
      this.subcategories = selectedCategory?.children || [];
      this.displayTable = false;
    },
    async getProperties() {
      this.selectedProperties = ["", "", ""];
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
        this.properties = response.data.data;
      } catch (error) {
        console.error(error);
      }
    },
    async fetchChildOptions(property, index) {
      this.displayTable = false;

      const selectedProperty = this.selectedProperties[index];
      if (property.name === "Brand" && selectedProperty !== "other") {
        try {
          const apiUrl = `${API_URL_OPTIONS_CHILD}${selectedProperty}`;
          const response = await axios.get(apiUrl, {
            headers: {
              "private-key": this.privateApiKey,
            },
          });
          this.options = response.data.data[0]?.options || [];
        } catch (error) {
          console.error(error);
        }
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

      this.properties.slice(0, 3).forEach((property, index) => {
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
          Value: (this.customOptionValue)?this.customOptionValue:this.selectedOption.name,
        });
      }

      this.displayTable = true; // Show the table
    },
  },
};
</script>
