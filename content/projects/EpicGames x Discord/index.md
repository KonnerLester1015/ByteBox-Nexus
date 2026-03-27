---
title: "Epic Games Free Games x Discord Webhook"
sidebar:
  exclude: true
---

## Overview

This project features an **n8n workflow** designed to bridge the gap between the Epic Games Store and Discord. While Epic Games provides a rotating selection of free titles, the metadata provided in their public API can sometimes be sparse. This workflow automatically scrapes the Epic Games Store for active promotions and then cross-references those titles with the **Steam API** to enrich the notifications with detailed game metadata, such as genres, original pricing, developer info, and install sizes.

By integrating both platforms, this automation provides a comprehensive "Free Game Alert" directly to a Discord channel, ensuring you and your friends never miss a high value claim.

{{< callout type="info" >}}
  Please note that while this workflow is designed around posting to Discord, the core logic for fetching and processing game data can be adapted to other platforms or notification systems with some modifications to the final notification step. Examples might be sending an email digest, posting to a different chat app, or even creating a custom dashboard of games over time... Endless possibilities!
{{< /callout >}}

### Workflow Overview

The workflow is designed for efficiency and to prevent unnecessary notifications in your Discord server. It includes an action step, **Extract Unique Games from the List**, which compares the current list of games with the list from the last successful run. A message is only sent when new free games are detected, ensuring that your Discord server is notified only when new content becomes available.

### Core Components

The workflow consists of the following key steps:

- **Schedule Trigger** – Automatically runs the workflow on a daily schedule.
- **Epic Games API Integration** – Retrieves current freeGamesPromotions data directly from Epic’s backend, including titles, promotional end dates, and thumbnails.
- **Parse Out Game Details** - Extracts relevant data from the API response, such as titles, promotional end dates, and thumbnail URLs.
- **Unique Game Filter** – Compares the current list of free games with previous n8n execution data to identify newly available titles.
- **Get Steam ID** – Queries the Steam API using the game title to retrieve its unique Steam ID.
- **Get Steam Details** – Uses the Steam ID to fetch additional metadata, including genres, original pricing, developer information, and install size.
- **Prepare notification** – Formats the collected data into a Discord webhook payload.
- **Notify Discord** – Sends the formatted notification to the configured Discord webhook URL.

Below is a visual overview of the workflow in n8n:

![Epic Games to Discord Workflow](/EpicGames-x-Discord-n8n-Flow.png)

---

## 1. Discord Webhook Setup

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

## 2. Workflow in n8n

The workflow runs on a daily schedule and performs the following section of steps:

{{< callout >}}
  Download the workflow JSON file from [My Github](https://github.com/KonnerLester1015/ByteBox-Nexus/blob/main/content/projects/EpicGames%20x%20Discord/EpicGames%20x%20Discord.json) and import it into your n8n instance.
{{< /callout >}}

### 2.1 Trigger

- **Schedule Trigger** – Configured to run everyday at 12 PM.

### 2.2 Call and Parse Epic Games Store

1. **HTTP Request**  
    - Makes a GET request to the Epic Games Store API endpoint for free game promotions.  
    - URL: `https://store-site-backend-static.ak.epicgames.com/freeGamesPromotions?locale=en-US&country=US&allowCountries=US`  
    - Returns a JSON response containing details about free game promotions.

2. **Parse Out Game Details**
    - JavaScript node that processes the API response to extract relevant details for each free game.  
    - The script navigates through the nested JSON structure to find the array of promoted games and extracts the title, description, original price, thumbnail image URL, store URL, and promotional dates.

```javascript {filename="JavaScript",linenos=table,linenostart=1}
// Function to format ISO date to readable EST string
// Example output: "Dec 18 at 11:00 AM EST"
function formatToEst(isoString) {
  if (!isoString) return "";
  
  const date = new Date(isoString);
  
  // Format options: "Dec 18", "11:00 AM"
  const options = { 
    timeZone: "America/New_York",
    month: "short", 
    day: "numeric", 
    hour: "numeric", 
    minute: "2-digit",
    hour12: true
  };
  
  // Create the string using standard JS localization
  // This automatically handles Daylight Savings (EDT vs EST)
  const formatted = date.toLocaleString("en-US", options);
  
  // Clean up the format to match your preferred style "Dec 18 at 11:00 AM EST"
  // .toLocaleString returns something like "Dec 18, 11:00 AM"
  return formatted.replace(",", " at") + " EST";
}

// --- Main Execution ---

// Get the list of games from the previous node's output
const allGames = items[0].json.data.Catalog.searchStore.elements;
const results = [];

for (const game of allGames) {
  // 1. Check if the discounted price is 0 (Free)
  // We check specifically for a discountPrice of 0 to avoid demos or free-to-play base games
  if (game.price.totalPrice.discountPrice === 0) {

    // 2. Filter out unwanted "Mystery Game" titles
    if (game.title.includes("Mystery Game")) {
      continue;
    }

    // 3. Get the "Tall" image (looks best on Discord/Mobile notifications)
    const imageObj = game.keyImages.find(img => img.type === 'OfferImageTall') || game.keyImages[0];

    // 4. Get the promo dates safely
    let startDateISO = "";
    let endDateISO = "";
    
    if (game.promotions && game.promotions.promotionalOffers && game.promotions.promotionalOffers.length > 0) {
        const offer = game.promotions.promotionalOffers[0].promotionalOffers[0];
        startDateISO = offer.startDate;
        endDateISO = offer.endDate;
    }

    // 5. Create the clean output item with converted dates
    results.push({
      json: {
        title: game.title,
        description: game.description,
        originalPrice: game.price.totalPrice.fmtPrice.originalPrice, 
        imageUrl: imageObj ? imageObj.url : "",
        storeUrl: `https://store.epicgames.com/en-US/p/${game.offerMappings[0].pageSlug}`,

        // Raw ISO dates (keep these if you need to sort by date later)
        startDateISO: startDateISO,
        endDateISO: endDateISO,
        
        // Format: "Dec 18 at 11:00 AM EST"
        startDate: formatToEst(startDateISO),
        endDate: formatToEst(endDateISO),
        
        // Combined string for notifications
        dateString: `Free Now - ${formatToEst(endDateISO)}` 
      }
    });
  }
}

return results;
```

Example output:

```json filename="JSON"
[
{
  "title": "Hyper Echelon",
  "description": "The Cyan Galaxy is in peril! Your trusty little star fighter and rag tag squad of wingmen are the only hope against the evil EXODON! Level up from a pea shooter to an offensive powerhouse as you engage in frantic combat across many wondrous locales.",
  "originalPrice": "$12.99",
  "imageUrl": "https://cdn1.epicgames.com/spt-assets/866f5f0834324794a87759b5f4854636/hyper-echelon-11w7f.png",
  "storeUrl": "https://store.epicgames.com/en-US/p/hyper-echelon-2d23ff",
  "startDateISO": "2026-03-26T15:00:00.000Z",
  "endDateISO": "2026-04-02T15:00:00.000Z",
  "startDate": "Mar 26 at 11:00 AM EST",
  "endDate": "Apr 2 at 11:00 AM EST",
  "dateString": "Free Now - Apr 2 at 11:00 AM EST"
},
{
  "title": "Hyper Echelon",
  "description": "The Cyan Galaxy is in peril! Your trusty little star fighter and rag tag squad of wingmen are the only hope against the evil EXODON! Level up from a pea shooter to an offensive powerhouse as you engage in frantic combat across many wondrous locales.",
  "originalPrice": "$12.99",
  "imageUrl": "https://cdn1.epicgames.com/spt-assets/866f5f0834324794a87759b5f4854636/hyper-echelon-11w7f.png",
  "storeUrl": "https://store.epicgames.com/en-US/p/hyper-echelon-2d23ff",
  "startDateISO": "2026-03-26T15:00:00.000Z",
  "endDateISO": "2026-04-02T15:00:00.000Z",
  "startDate": "Mar 26 at 11:00 AM EST",
  "endDate": "Apr 2 at 11:00 AM EST",
  "dateString": "Free Now - Apr 2 at 11:00 AM EST"
}
]
```

---

### 2.3 Change Detection (Unique Game Filter)

**Extract Unique Games from the List**  
- Runs JavaScript to hash the list of titles. 
- Compares against data from the last workflow execution.  
- Returns the list of new games if unique entries are found

 More information on this approach and the `getWorkflowStaticData` method can be found in the [n8n documentation](https://docs.n8n.io/code/cookbook/builtin/get-workflow-static-data/).

```javascript {filename="JavaScript",linenos=table,linenostart=1}
 // Get a reference to the static data for this node, which persists across executions.
const nodeStaticData = $getWorkflowStaticData('node');

// Get all the items from the previous node.
const allGames = $input.all();

// Get the history of games that have already been processed.
// Use a Set for faster lookups.
const gameHistory = new Set(nodeStaticData.gameHistory || []);

// Filter the games to find only the new ones.
// Skip if title is empty.
const newGamesToPost = allGames.filter(item => {
    const title = item.json.title;

    // Skip if title is "" or null/undefined
    if (!title || title.trim() === "") {
        return false;
    }

    // Otherwise check if it's not in history
    return !gameHistory.has(title);
});

// If there are new games to post, update the history.
if (newGamesToPost.length > 0) {
    const newGameTitles = newGamesToPost.map(item => item.json.title);
    newGameTitles.forEach(title => gameHistory.add(title));
    
    nodeStaticData.gameHistory = [...gameHistory];
}

// Return the list of games that are new.
return newGamesToPost;
```

---

### 2.4 Get Steam Metadata
1. **Get Steam ID**  
   - JavaScript node that takes the game title and queries the Steam API to find the corresponding Steam App ID.
   - Makes a POST request to `https://store.steampowered.com/api/storesearch?term=GAME_TITLE&l=english&cc=US` endpoint, where `GAME_TITLE` is the title of the game from the Epic Games Store.

2. **Does SteamID Exist**
   - Conditional node that checks if a Steam App ID was returned from the previous step (*json.total is not equal to 0*)
   - If a Steam ID exists, the workflow continues to the next step to fetch detailed metadata. If not, it skips to the notification preparation step with only Epic Games data.

3. **Get Steam Details**
    - HTTP Request node that takes the Steam App ID and queries the Steam API for detailed metadata about the game.
    - Makes a GET request to `https://store.steampowered.com/api/appdetails?appids=STEAM_APP_ID&l=english&cc=US` endpoint, where `STEAM_APP_ID` is the ID retrieved from the previous step.
    - Extracts relevant metadata such as genres, original pricing, developer info, and install sizes to enrich the Discord notification.

Example Steam metadata output:

{{< callout type="warning" >}}
  Note that the below example response is shortened for brevity. The actual response from the Steam API contains much more detailed information about the game including required_age, a detailed description, controller support, supported languages, and more. You can choose which fields to include in your Discord notification based on your preferences.
{{< /callout >}}

```json {filename="JavaScript"}
{
  "953330": {
    "success": true,
    "data": {
      "name": "Hyper Echelon",
      "steam_appid": 953330,
      "short_description": "The Cyan Galaxy is in peril! Your trusty little star fighter and rag tag squad of wingmen are the only hope against the evil EXODON!",
      "genres": [
        {
          "id": "1",
          "description": "Action"
        },
        {
          "id": "23",
          "description": "Indie"
        }
      ],
      "price_overview": {
        "currency": "USD",
        "final_formatted": "$12.99"
      },
      "developers": ["GangoGames LLC"],
      "header_image": "https://shared.akamai.steamstatic.com/store_item_assets/steam/apps/953330/header.jpg"
    }
  }
}
```

### 2.5 Notification Handling

1. **Prepare Notification**
- JavaScript node that takes the combined data from both the Epic Games Store and Steam API to format a rich embed message for Discord. The script constructs a payload that includes the game title, description, genres, Price, release date, developer, game modes, install size, and promotional end date. It also includes the game's thumbnail image and a link to the store page.
- The payload utilzes Discord support for embeds to create a visually appealing message that stands out in the channel compared to plain text notifications.
- The script also includes fallback logic to ensure that if certain metadata is missing from either source, the notification still contains useful information without breaking the formatting.
- The payload is structured to include a username and avatar for the webhook, making it clear that the message is coming from the "Free Game Alert" bot or whatever name you choose.

{{< callout type="info" >}}
  Refer to the Discord Developer documentation on [Webhook Embeds](https://discord.com/developers/docs/resources/webhook#embed-object) for more details on how to customize the appearance of your notifications with different embed fields, colors, and formatting options.
{{< /callout >}}

```javascript {filename="JavaScript",linenos=table,linenostart=1}
// Helper: Extract Storage from HTML
function getInstallSize(reqsHtml) {
    if (!reqsHtml) return "Unknown";

    // Match the Storage field inside <strong> tags
    const match = reqsHtml.match(
        /<strong>\s*(?:Storage|Hard Drive|Space):\s*<\/strong>\s*([^<]+)/i
    );

    if (match && match[1]) {
        // Extract only the size value (e.g., "75 GB", "120 MB")
        const sizeMatch = match[1].match(/\d+(?:\.\d+)?\s*(?:KB|MB|GB|TB)/i);
        if (sizeMatch) {
            return `~${sizeMatch[0]}`;
        }
    }

    return "Unknown";
}

// Initialize empty array for embeds
let embeds = [];

// Get all items from the current input (Steam Details)
const items = $input.all();

// Access the original Epic Games data from the previous node
const epicItems = $('Loop Over Items').all();

for (let i = 0; i < items.length; i++) {
    const item = items[i];
    
    // --- 1. GET EPIC DATA ---
    const epicData = epicItems[i] ? epicItems[i].json : {};
    
    const title = epicData.title || "";
    const storeUrl = epicData.storeUrl || "https://store.epicgames.com";
    const imageUrl = epicData.imageUrl || "";
    const endDate = epicData.endDate || "Limited Time";
    const epicDescription = epicData.description || "";

    // --- 2. EXTRACT STEAM DATA ---
    let genres = "Unknown";
    let originalPrice = "Unknown";
    let developers = "Unknown";
    let releaseDate = "Unknown";
    let gameModes = "Unknown";
    let installSize = "Unknown"; 
    let steamDescription = "";
    
    const steamKey = Object.keys(item.json)[0];
    
    if (steamKey && item.json[steamKey].success) {
        const game = item.json[steamKey].data;
        
        // A. Developers
        if (game.developers && game.developers.length > 0) {
            developers = game.developers.join(", ");
        }
        
        // B. Genres
        if (game.genres) {
            genres = game.genres.map(g => g.description).slice(0, 3).join(", ");
        }
        
        // C. Price
        if (game.price_overview) {
            if (game.price_overview.initial_formatted) {
                originalPrice = `~~${game.price_overview.initial_formatted}~~`;
            } else if (game.price_overview.initial > 0) {
                originalPrice = `~~$${(game.price_overview.initial / 100).toFixed(2)}~~`;
            }
        } else if (game.is_free) {
            originalPrice = "Free To Play";
        }
        
        // D. Release Date
        if (game.release_date) {
            releaseDate = game.release_date.date;
        }

        // E. Game Modes
        if (game.categories) {
            const relevantModes = game.categories
                .map(c => c.description)
                .filter(desc => 
                    desc === "Single-player" || 
                    desc === "Multi-player" || 
                    desc === "Co-op" || 
                    desc === "Online Co-op" ||
                    desc === "PvP" || 
                    desc === "Shared/Split Screen"
                );
            
            if (relevantModes.length > 0) {
                gameModes = [...new Set(relevantModes)].join(" / ");
            }
        }

        // F. Install Size
        if (game.pc_requirements) {
            // Check minimum requirements first
            installSize = getInstallSize(game.pc_requirements.minimum);
            // If not found, check recommended
            if (installSize === "Unknown" && game.pc_requirements.recommended) {
                installSize = getInstallSize(game.pc_requirements.recommended);
            }
        }
        
        // G. Steam Description (for fallback)
        steamDescription = game.short_description;
    }

    // --- 3. BUILD DISCORD EMBED ---
    var content = {
        "author": {
            "name": "Free Game Alert",
            "icon_url": "https://upload.wikimedia.org/wikipedia/commons/thumb/3/31/Epic_Games_logo.svg/1200px-Epic_Games_logo.svg.png"
        },
        "title": title,
        "url": storeUrl,
        "color": 30962, 
        // Use Epic Description first, fallback to Steam, then generic default
        "description": epicDescription || steamDescription || "Check store for details.",
        
        "thumbnail": {
            "url": imageUrl
        },
        "fields": [
            // Row 1
            { "name": "Genre", "value": genres, "inline": true },
            { "name": "Price", "value": originalPrice, "inline": true },
            { "name": "Release Date", "value": releaseDate, "inline": true },
            
            // Row 2
            { "name": "Developer", "value": developers, "inline": true },
            { "name": "Game Mode", "value": gameModes, "inline": true },
            { "name": "Size", "value": installSize, "inline": true },
            
            // Row 3
            { "name": "Offer Ends", "value": endDate, "inline": false }
        ],
        "footer": {
            "text": "Epic Games Store • Steam Metadata",
            "icon_url": "https://upload.wikimedia.org/wikipedia/commons/thumb/3/31/Epic_Games_logo.svg/1200px-Epic_Games_logo.svg.png"
        }
    };
    
    embeds.push(content);
}

const payload = {
    username: "Free Game Alert",
    avatar_url: "https://upload.wikimedia.org/wikipedia/commons/6/6c/Input-gaming.png",
    embeds: embeds
};

return [{json: {body: payload}}];
```

2. **Notify Discord**  
  - HTTP Request node that will take the formatted payload from the previous step and send it to the Discord webhook URL using a POST request.

---

## Outcome

The result of this workflow is a rich, informative Discord message that alerts you to new free games available on the Epic Games Store, complete with detailed metadata from Steam when available. This allows you and your friends to quickly see which games are free, what they are about, and when the promotion ends, all without having to manually check the store.

- **Automated Notifications** – Daily schedule ensures you never miss a free game.  
- **Rich Metadata** – Steam API integration provides detailed game info beyond what Epic's API offers.  
- **Unique Game Detection** – Only new free games trigger notifications, preventing spam.
- **No Need for Authentication Keys** - Both the Epic Games Store and Steam APIs used in this workflow are public endpoints that do not require authentication keys for the specific data being accessed. This allows for a straightforward setup without the need to manage API credentials.

Example Discord message:
![Discord Notification Example](/DiscordWebhookMessage.png)