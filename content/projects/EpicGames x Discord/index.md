---
title: "Epic Games Free Games x Discord Webhook"
sidebar:
  exclude: true
---

## Overview

This project describes how to build an **n8n workflow** that scrapes the Epic Games Store’s free game promotions and posts them to a Discord channel via webhook.  

Epic Games often offers paid games for free for a limited time, and this workflow ensures you and your friends are automatically notified.  

The implementation uses the **Browserless.io** service to bypass bot detection and to fetch HTML content from the Epic Games Store, parses the results with n8n’s HTML and scripting nodes, and finally formats and posts messages to Discord. It is based on a community workflow but adapted to replace the Puppeteer node with Browserless.

### Workflow Overview

The workflow is designed to be efficient and to prevent spamming your Discord server. It includes a **Check if Changed** step that compares the current list of games to the last successful run's list. A message is only sent if new free games are detected, ensuring that your discord server is only notified of new content.

### Core Components

The workflow consists of the following key steps:

- **Schedule Trigger** – Runs the workflow automatically on a weekly schedule.
- **HTTP Request** – Uses the Browserless.io API to get the HTML content from the Epic Games Store.
- **HTML & Split Out Nodes** – Parse the HTML content to extract individual game containers and then split them into separate items for processing.
- **Extract Title and Image** – Extracts specific details like the game's title, image URL, and free dates.
- **Check if Changed** – Compares the current games to a saved list from the last run.
- **Prepare notification** – Formats the game details into a Discord webhook payload.
- **Notify Discord** – Sends the formatted message to your Discord webhook URL.

---

## 1. Browserless.io Setup

This workflow uses Browserless.io's BQL (Browserless Query Language) to scrape the Epic Games Store page, ensuring it is not blocked by Cloudflare's bot protection. The query navigates to the Epic Games Store URL, waits for specific selectors, and extracts HTML content from the free game section.  

1. Go to [https://www.browserless.io/](https://www.browserless.io/).  
2. Create an account and copy your API key.  
3. Use the **BQL Editor** to generate a cURL request for the Epic Games store.  
   Example cURL request (This will be imported directly into n8n's HTTP Request node.):

   ```bash
   curl --request POST \
     --url 'https://production-sfo.browserless.io/chromium/bql?token=YOUR_API_TOKEN_HERE&proxy=residential' \
     --header 'Content-Type: application/json' \
     --data '{
       "query": "mutation VerifyEpicGamesCloudflare {\n  goto(url: \"https://store.epicgames.com/en-US/\", waitUntil: networkIdle) {\n    status\n  }\n  waitForSelector(selector: \".cf-turnstile\", timeout: 1000) {\n    time\n  }\n  if(selector: \".cf-turnstile\") {\n    verify(type: cloudflare, timeout: 30000) {\n      found\n      solved\n      time\n    }\n    waitForNavigation(waitUntil: networkIdle, timeout: 20000) {\n      time\n    }\n  }\n  html(selector: \".css-cdosd6\") {\n    html\n  }\n}",
       "variables": {},
       "operationName": "VerifyEpicGamesCloudflare"
     }'
    ```

## 2. Discord Webhook Setup

Before n8n can send messages to Discord, you need to create a webhook in your Discord server:

1. Open **Discord** and right-click on the server icon/name.  
2. Select **Server Settings** → **Integrations**.  
3. Under **Webhooks**, click **Create Webhook**.  
4. Configure:
   - **Name** – e.g., *Epic Games Bot*  
   - **Avatar** – optional bot profile image  
   - **Channel** – where notifications should post  
5. Click **Copy Webhook URL**.  

Keep this URL safe — it will be used in the n8n workflow to send messages.

---

## 3. Workflow in n8n

The workflow, titled **EpicGames x Discord**, runs on a weekly schedule and performs the following steps:

Below is a visual overview of the workflow in n8n:

![Epic Games to Discord Workflow](/n8n-EpicxDiscordv2.png)

Download the workflow JSON file from [My Github](https://github.com/KonnerLester1015/ByteBox-Nexus/blob/main/public/images/EpicGames%20x%20Discord.json) and import it into your n8n instance.

### 3.1 Trigger

- **Schedule Trigger** – Configured to run every Monday at 12 PM.

### 3.2 Scraping and Extraction

1. **HTTP Request**  
    *This is where we import the cURL that we created in the BQL Editor*
   - Sends a POST request to Browserless.io with a GraphQL query to load the Epic Games Store.  
   - Handles Cloudflare verification if needed and returns the raw HTML of the store page.

2. **HTML - Extract Free Games Container**  
   - Uses n8n’s HTML node to extract game offer containers.  
   - CSS selector:  
     `
     [data-component*="OfferCard"]:not([data-component="DiscoverOfferCard"])
     `
   - Returns an array of HTML blocks for each free game.

3. **Split Out Each Container (Game)**  
   - Splits the container array into individual items (one per game).

4. **Extract Title and Image**  
   - Extracts details for each game:  

     | Key   | CSS Selector | Return Value | Attribute   |
     |-------|--------------|--------------|-------------|
     | title | h6           | Text         |             |
     | image | img          | Attribute    | data-image  |
     | url   | a            | Attribute    | href        |
     | dates | p > span     | Text         |             |

    Example output:

    ```json
    [
      {
        "title": "Monument Valley",
        "image": "https://cdn1.epicgames.com/spt-assets/e56a7411805046d3b5b7253a6e4e0faa/monument-valley-bp09k.jpg?resize=1&w=360&h=480&quality=medium",
        "url": "/en-US/p/monument-valley-1d99d3",
        "dates": "Free Now - Sep 11 at 08:00 AM"
      },
      {
        "title": "Ghostrunner 2",
        "image": "https://cdn1.epicgames.com/offer/708f57aaa04b42ef885be16c8288f0ac/EGS_Ghostrunner2_OneMoreLevel_S2_1200x1600-12ae7597b946ad94ac75680eb735be07?resize=1&w=360&h=480&quality=medium",
        "url": "/en-US/p/ghostrunner-2",
        "dates": "Free Sep 11 - Sep 18"
      },
      {
        "title": "Monument Valley 2",
        "image": "https://cdn1.epicgames.com/spt-assets/ec6e53ce95cc432d895dfb1285254bfd/monument-valley-2-1xlny.jpg?resize=1&w=360&h=480&quality=medium",
        "url": "/en-US/p/monument-valley-2-addd02",
        "dates": "Free Sep 11 - Sep 18"
      },
      {
        "title": "The Battle of Polytopia",
        "image": "https://cdn1.epicgames.com/spt-assets/b312dfc1b33d459daa55a67d53a56223/the-battle-of-polytopia-1l3xz.png?resize=1&w=360&h=480&quality=medium",
        "url": "/en-US/p/the-battle-of-polytopia-12fed6",
        "dates": "Free Sep 11 - Sep 18"
      }
    ]
    ```

---

### 3.3 Change Detection

5. **Check if Changed**  
   - Runs custom JavaScript to hash the list of titles.  
   - Compares against static data from the workflow last execution.  
   - Returns the list of new games if changes are detected, otherwise returns no items.

    More information on this approach and the `getWorkflowStaticData` method can be found in the [n8n documentation](https://docs.n8n.io/code/cookbook/builtin/get-workflow-static-data/).

    ```javascript
    // Get a reference to the static data for this node, which persists across executions.
    const nodeStaticData = $getWorkflowStaticData('node');

    // Get all the items from the previous node.
    const allGames = $input.all();

    // Get the history of games that have already been processed.
    // Use a Set for faster lookups.
    const gameHistory = new Set(nodeStaticData.gameHistory || []);

    // Filter the games to find only the new ones.
    // We'll return the full item object, not just the title.
    const newGamesToPost = allGames.filter(item => {
        // Check if the game's title is NOT in our history Set.
        return !gameHistory.has(item.json.title);
    });

    // If there are new games to post, update the history.
    if (newGamesToPost.length > 0) {
        // Get the titles of the new games.
        const newGameTitles = newGamesToPost.map(item => item.json.title);
        
        // Add all the new game titles to the history.
        newGameTitles.forEach(title => gameHistory.add(title));
        
        // Store the updated history back in the static data.
        nodeStaticData.gameHistory = [...gameHistory];
    }

    // Return the list of games that are new.
    return newGamesToPost;
    ```

---

### 3.4 Notification Handling

6. **Convert PST to EST**
    - JavaScript node that adjusts the free game dates from Pacific Time to Eastern Time for accurate Discord posting.
  
      ```javascript
      function convertPstToEst(dateString) {
        // Regex to capture the prefix (e.g., "Free Now - Oct 23 at"), hours, minutes, and AM/PM
        // The regex ensures a full time string (HH:MM AM/PM) is present.
        const regex = /^(.*?)\s*(\d{1,2}):(\d{2})\s+(AM|PM)$/i;
        const match = dateString.match(regex);
        
        // If the format doesn't match (e.g., "Free Oct 23 - Oct 30"), return the original string immediately.
        if (!match) {
            return dateString;
        }

        // Extract components
        const prefix = match[1].trim(); // Everything before the time
        let hours = parseInt(match[2], 10);
        const minutes = match[3];
        const period = match[4].toUpperCase();

        // 1. Convert 12-hour to 24-hour
        let hours24 = hours;
        if (period === 'PM' && hours < 12) {
            hours24 += 12;
        } else if (period === 'AM' && hours === 12) {
            hours24 = 0; // Midnight (12 AM) is hour 0
        }

        // 2. Add 3 hours for the time zone shift (PST/PDT to EST/EDT)
        let newHours24 = hours24 + 3;

        // 3. Handle 24-hour rollover (e.g., 23 + 3 = 26, 26 % 24 = 2)
        // Note: This correctly adjusts the hour but does not change the day/date string.
        newHours24 %= 24;

        // 4. Convert 24-hour back to 12-hour format
        let newPeriod = (newHours24 >= 12) ? 'PM' : 'AM';
        let newHours12 = newHours24 % 12;
        if (newHours12 === 0) {
            newHours12 = 12; // 0 (midnight) or 12 (noon) becomes 12
        }

        // Ensure minutes maintain two digits
        const newTime = `${newHours12}:${minutes} ${newPeriod}`;

        // 5. Return the final string with the 'EST' label
        return `${prefix} ${newTime} EST`;
      }

      // --- Main Script Execution ---

      // Loop over input items and convert time zone for applicable strings, updating the 'dates' field.
      for (const item of $input.all()) {
          // Use the existing dates field as the input string
          const datesString = item.json.dates;

        // Process only if the dates field exists
          if (datesString) {
              // The function will convert the time if found, or return the original string if no time is present.
              item.json.dates = convertPstToEst(datesString);
          }
          // If datesString is missing, the field remains unchanged (null/undefined).
      }
      return $input.all();
      ```

7. **Prepare notification**  
   - JavaScript node that formats the data in preparation of the Discord webhook payload.  
   - Adds embeds for each game with:  
     - Title  
     - URL (Epic Games Store link)  
     - Thumbnail (game image)  
     - Description (dates the game is free)  
   - Includes a custom message and avatar for the bot.

    ```javascript
    // Initialize an empty array to store the embed objects for the Discord message.
    let embeds = [];

    // Loop through each game received from the workflow's input.
    for (const item of $input.all()) {
        // Create a base content object for the embed, starting with the thumbnail.
        var content = {
            "thumbnail": {
                // The thumbnail URL is taken from the 'image' property of the current input item's JSON.
                "url": item.json.image
            }
        };

          // Add the title, URL, and a description to the content object.
          content["title"] = item.json.title;
          // The URL is constructed by prepending the Epic Games Store base URL.
          content["url"] = "https://store.epicgames.com" + item.json.url,
          // The description is taken from the 'dates' property.
          content["description"] = item.json.dates;

        // Add the newly created content object (which is a Discord embed) to the embeds array.
        embeds.push(content);
    }

    // Create the final payload object for the Discord webhook.
    const payload = {
    // Set the display name for the bot.
    username: "Epic Games",
    // Set the main text content of the message.
    content: "New Free Game(s) from Epic Games Store",
    // Set the avatar URL for the bot's profile.
    avatar_url: "https://upload.wikimedia.org/wikipedia/commons/thumb/3/31/Epic_Games_logo.svg/250px-Epic_Games_logo.svg.png",
    // Attach the array of embeds to the payload.
    embeds: embeds
    };

    // Return the final payload as a JSON object within an array, so it can be used by the next node in the workflow (e.g., a Discord webhook node).
    return [{json: {body: payload}}];
    ```

8. **Notify Discord**  
   - HTTP Request node posts the formatted payload to the configured Discord webhook URL.

---

## Outcome

The workflow runs automatically every Monday at 12 PM. It scrapes the Epic Games Store, checks if the list of free games has changed, and posts a formatted Discord message with game details. If no changes are detected, no message is sent keeping the Discord server free from unnecessary spam.

This project delivers a reliable, low-maintenance way to stay updated on Epic Games Store promotions with friends.

- **Automated Notifications** – Weekly schedule ensures you never miss a free game.  
- **Efficient Scraping** – Browserless.io handles site access and bypasses bot detection.  
- **Spam Prevention** – Change detection logic sends updates only when new games appear.  
- **Clear Messaging** – Each Discord post includes the game’s title, image, link, and free-to-claim dates in rich embed format.

Example Discord message:
![Discord Notification Example](/DiscordWebhookMessage.png)