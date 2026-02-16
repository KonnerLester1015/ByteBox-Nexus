---
title: "Weather CLI Project"
sidebar:
  exclude: true
---
## Introduction

This project involves the development of a **console-based weather application** that uses the **Weatherstack API** to fetch and display real-time weather data. The primary goal is to enable users to input a city location and receive detailed weather information, including **UV index, temperature, humidity, wind speed**, and more. Designed with accessibility and ease of use in mind, this project offers an efficient tool for users to check weather conditions for any specified location.

> Note: This project was part of a school assignment for my Systems Programming class.  

## Objectives

The main objectives for this project are:

**Data Accessibility:** Offer real-time weather information based on a location’s identifiable information with an easy-to-use user interface.

**Error Management:** Implement robust error handling to manage invalid inputs and unsuccessful API calls, enhancing reliability and providing user feedback.

**API Integration:** Use the Weatherstack API along with the libcurl and jsmn libraries for API communication and JSON parsing.

## Program Architecture

The program code is structured modularly, making it manageable and allowing each component to serve as a building block for the entire program. The main components are:

- **Input Block:** This block will prompt and collect user input (e.g. city name) and include necessary code for validating the user’s input.
- **API Block:** This block of code will establish the connection with the Weatherstack API by sending an HTTP request and handling the response with the help of the libcurl library.
- **Data Processing:** This block will contain the code for parsing the API response which will be in a JSON format. This code will utilize the **jsmn** library, or similar, to organize the relevant data into a C data structure.
- **Output/Display Block:** This block will take the data structure and will display the parsed data to the user in a console friendly format.

Additionally, the code will be compiled with the **gcc compiler**.

## Program Flow

The program will first welcome the user and introduce the weather functionality (e.g. offer an example input). Then the user will provide an input. The program will validate the input and if valid will make an API call to retrieve the weather data for that location. The response to the call will then be parsed and presented to the user. If input is invalid or if the call fails, the program will provide error handling and prompt the user to retry.

## Procedure

1. Create WeatherStack Account
   - Navigate to <https://weatherstack.com/> and create an account. Once the account is made you will be presented with your API key
2. Install libcurl

   ```bash {filename="bash"}
   sudo apt-get install libcurl4-openssl-dev
   ```

3. Download jsmn.h (Used for JSON parsing)
   - Download the jsmn.h file from <https://github.com/zserge/jsmn/blob/master/jsmn.h>

4. Compile code

   ```bash {filename="bash"}
   sudo gcc -o weather_cli main.c -lcurl
   ```

## Future Improvements

**Graphical User Interface:**

  - Enhance accessibility by introducing a GUI with visual weather elements like icons.

**Historical and Forecast Data:**

   - Include features to display historical data and multi-day forecasts.

**Error Handling:**

   - Improve handling of API request failures, invalid inputs, and network issues.

## C Source Code

{{% details title="View Code" closed="true" %}}

```C {linenos=table,linenostart=1,filename="WeatherStack.c"}
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <curl/curl.h>
#include "jsmn.h"

// Constants for maximum input sizes
#define MAXCITYSIZE 102
#define MAXZIPSIZE 7
#define MAXCOORDSIZE 12
#define MAXIPSIZE 17

// Structure to store response data dynamically
struct MemoryStruct {
    char *memory; // Pointer to the memory where the response data will be stored
    size_t size;  // Size of the memory allocated
};

// Callback function to write response data into memory
static size_t WriteCallback(void *contents, size_t size, size_t nmemb, void *userp) {
    size_t realsize = size * nmemb; // Calculate the real size of the incoming data
    struct MemoryStruct *mem = (struct MemoryStruct *)userp;

    // Reallocate memory to accommodate the new data
    char *ptr = realloc(mem->memory, mem->size + realsize + 1);

    mem->memory = ptr; // Update the memory pointer
    memcpy(&(mem->memory[mem->size]), contents, realsize); // Copy the new data into the memory
    mem->size += realsize; // Update the size of the memory
    mem->memory[mem->size] = 0; // Null-terminate the string

    return realsize;
}

// Helper function to find a JSON key and get its value
const char *get_json_value(const char *json, jsmntok_t *tokens, int num_tokens, const char *key) {
    for (int i = 0; i < num_tokens - 1; i++) {
        if (tokens[i].type == JSMN_STRING && 
            strncmp(json + tokens[i].start, key, tokens[i].end - tokens[i].start) == 0) {
            // Return the value associated with the key as a new string
            return strndup(json + tokens[i + 1].start, tokens[i + 1].end - tokens[i + 1].start);
        }
    }
    return NULL; // Return NULL if the key is not found
}

// Generate the weather output (https://github.com/zserge/jsmn/tree/master)
void generateWeatherParagraph(const char *jsonResponse, char unit) {
    jsmn_parser parser;
    jsmntok_t tokens[256];
    jsmn_init(&parser);

    int token_count = jsmn_parse(&parser, jsonResponse, strlen(jsonResponse), tokens, 256);

    // Extract weather details
    const char *city = get_json_value(jsonResponse, tokens, token_count, "name");
    const char *region = get_json_value(jsonResponse, tokens, token_count, "region");
    const char *country = get_json_value(jsonResponse, tokens, token_count, "country");
    const char *localtime = get_json_value(jsonResponse, tokens, token_count, "localtime");
    const char *temperature = get_json_value(jsonResponse, tokens, token_count, "temperature");
    const char *weather_desc_raw = get_json_value(jsonResponse, tokens, token_count, "weather_descriptions");
    // Clean up the raw weather description by removing brackets
    const char *weather_desc = weather_desc_raw ? strndup(weather_desc_raw + 2, strlen(weather_desc_raw) - 4) : "N/A";
    const char *wind_dir = get_json_value(jsonResponse, tokens, token_count, "wind_dir");
    const char *wind_speed = get_json_value(jsonResponse, tokens, token_count, "wind_speed");
    const char *humidity = get_json_value(jsonResponse, tokens, token_count, "humidity");
    const char *uv_index = get_json_value(jsonResponse, tokens, token_count, "uv_index");
    const char *visibility = get_json_value(jsonResponse, tokens, token_count, "visibility");
    const char *pressure = get_json_value(jsonResponse, tokens, token_count, "pressure");
    const char *precipitation = get_json_value(jsonResponse, tokens, token_count, "precip");

    // Determine the unit system and appropriate labels
    const char *unit_system;
    const char *temp_unit;
    const char *precip_unit = "mm";
    const char *wind_unit = "km/h";
    const char *visibility_unit = "km";
    if (unit == 'M') {
        unit_system = "Metric system";
        temp_unit = "Celsius";
    } else if (unit == 'S') {
        unit_system = "Scientific system";
        temp_unit = "Kelvin";
    } else {
        unit_system = "Fahrenheit system";
        temp_unit = "Fahrenheit";
        precip_unit = "in";
        wind_unit = "mph";
        visibility_unit = "miles";
    }

    // Determine the UV index scale description
    const char *uv_scale_description = "N/A";
    if (uv_index) {
        int uv_value = atoi(uv_index);
        if (uv_value <= 2) {
            uv_scale_description = "Low";
        } else if (uv_value <= 5) {
            uv_scale_description = "Moderate";
        } else if (uv_value <= 7) {
            uv_scale_description = "High";
        } else if (uv_value <= 10) {
            uv_scale_description = "Very High";
        } else {
            uv_scale_description = "Extreme";
        }
    }

    // Generate the weather paragraph with updated sentence structure
    printf("\n==========================================\n");
    printf("           Weather Update\n");
    printf("==========================================\n\n");
    printf("The weather in the city of %s, in the region of %s, %s, is currently %s degrees %s with %s skies.\n", 
            city, region, country, temperature, temp_unit, weather_desc);
    printf("With a UV index of %s (%s exposure risk).\n", uv_index, uv_scale_description);
    printf("The wind is blowing from the %s at %s %s with a humidity of %s percent.\n", 
            wind_dir, wind_speed, wind_unit, humidity);
    printf("Visibility is at %s %s, and the atmospheric pressure is %s hPa.\n", 
            visibility, visibility_unit, pressure);
    printf("Precipitation is at %s %s.\n", precipitation, precip_unit);
    printf("The local time is %s.\n", localtime);

    printf("\n==========================================\n\n");

    // Free dynamically allocated strings
    free((void *)city);
    free((void *)region);
    free((void *)country);
    free((void *)localtime);
    free((void *)temperature);
    free((void *)weather_desc_raw);
    free((void *)weather_desc);
    free((void *)wind_dir);
    free((void *)wind_speed);
    free((void *)humidity);
    free((void *)visibility);
    free((void *)pressure);
    free((void *)precipitation);
    free((void *)uv_index);
}


// Function to get user input for the unit
char getUnitInput() {
    char unit;
    printf("Choose unit (M for Metric, S for Scientific, F for Fahrenheit): ");
    scanf(" %c", &unit);
    switch (unit) {
        case 'M': case 'm': return 'M';
        case 'S': case 's': return 'S';
        case 'F': case 'f': return 'F';
        default: 
            printf("Invalid input, defaulting to Fahrenheit.\n");
            return 'F';
    }
}

// Function to get a string input from the user
void getStringInput(char *prompt, char *input, int maxSize) {
    printf("%s", prompt);
    fgets(input, maxSize, stdin);
    // Remove newline character
    input[strcspn(input, "\n")] = '\0';
}

// Function to make a CURL request (using libcurl - https://curl.se/libcurl/)
void makeRequest(const char *access_key, const char *location, char unit, struct MemoryStruct *response) {
    CURL *curl = curl_easy_init();

    char *encoded_location = curl_easy_escape(curl, location, 0);
    char url[256];
    snprintf(url, sizeof(url), "http://api.weatherstack.com/current?access_key=%s&query=%s&units=%c", 
             access_key, encoded_location, unit);

    curl_easy_setopt(curl, CURLOPT_URL, url);
    curl_easy_setopt(curl, CURLOPT_WRITEFUNCTION, WriteCallback);
    curl_easy_setopt(curl, CURLOPT_WRITEDATA, (void *)response);

    CURLcode res = curl_easy_perform(curl);
    if (res != CURLE_OK) {
        fprintf(stderr, "CURL error: %s\n", curl_easy_strerror(res));
    } else {
        generateWeatherParagraph(response->memory, unit);
    }

    curl_free(encoded_location);
    curl_easy_cleanup(curl);
}

int main() {
    const char *access_key = //Add WeatherStack API token here
    struct MemoryStruct response = { .memory = malloc(1), .size = 0 };

    //Set default search
    char location[MAXCITYSIZE], unit; 
    int choice;

    do {
        // Print User Menu
        printf("\nWeather Program Menu:\n");
        printf("1 - Enter City\n");
        printf("2 - Enter Zipcode\n");
        printf("3 - Enter Coordinates\n");
        printf("4 - Enter IP Address\n");
        printf("5 - Exit\n");
        printf("Your choice: ");
        scanf("%d", &choice);
        getchar();

        switch (choice) {
            case 1:
                getStringInput("Enter a city: ", location, MAXCITYSIZE);
                unit = getUnitInput();
                break;
            case 2: {
                char zip[MAXZIPSIZE];
                getStringInput("Enter a 5-digit zipcode: ", zip, MAXZIPSIZE);
                snprintf(location, MAXCITYSIZE, "%s", zip);
                unit = getUnitInput();
                break;
            }
            case 3: {
                char lat[MAXCOORDSIZE], lon[MAXCOORDSIZE];
                getStringInput("Enter latitude: ", lat, MAXCOORDSIZE);
                getStringInput("Enter longitude: ", lon, MAXCOORDSIZE);
                snprintf(location, MAXCITYSIZE, "%s,%s", lat, lon);
                unit = getUnitInput();
                break;
            }
            case 4: {
                char ip[MAXIPSIZE];
                getStringInput("Enter an IP address: ", ip, MAXIPSIZE);
                snprintf(location, MAXCITYSIZE, "%s", ip);
                unit = getUnitInput();
                break;
            }
            case 5:
                printf("Exiting program. Goodbye!\n");
                break;
            default:
                printf("Invalid option. Please try again.\n");
                break;
        }

        if (choice >= 1 && choice <= 4) {
            // Clear response memory before making a new request
            free(response.memory);
            response.memory = malloc(1); // Reallocate memory
            response.size = 0;

            curl_global_init(CURL_GLOBAL_DEFAULT);
            //Create API request
            makeRequest(access_key, location, unit, &response);
            curl_global_cleanup();
        }

    } while (choice != 5);

    free(response.memory);
    return 0;
}
```

{{% /details %}}

## Demo Video

{{< youtube _IB6kkw6uhU >}}
