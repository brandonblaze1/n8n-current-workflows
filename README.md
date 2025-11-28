Blaze Social Media Video Publishing Pipeline

Automated Cross-Platform Publishing for Rental Listing Videos
Workflow: publish-videos
System: n8n + Google Drive + NocoDB + OpenAI + Blotato
Source: publish-videos.json 

publish-videos

Overview

This workflow automates the final 80% of your property-listing video distribution process:

Generates platform-specific marketing text using OpenAI.

Saves all ad copy to NocoDB.

Reads the finalized video exports from Google Drive.

Identifies platform versions (TikTok / FB Reel / FB Portrait / YT Short / YT Full) by filename.

Uploads each video to Blotato.

Publishes to Facebook + YouTube.

Writes all published post IDs and media URLs back into NocoDB for record-keeping.

Manual steps remaining:

Create the property listing details (Title, Description, Rent, Beds/Baths, Sqft).

Export the videos to Google Drive using the correct filenames and suffixes.

Everything after that? Automatic.

File Naming Convention

Place the following files in your Google Drive folder:

/social-media-videos/
    SLUG.txt
    SLUG-fb-prt.mp4
    SLUG-fb-reel.mp4
    SLUG-tiktok.mp4
    SLUG-yt-short.mp4
    SLUG-yt.mp4


Examples:

2024-oak.txt
2024-oak-fb-prt.mp4
2024-oak-fb-reel.mp4
2024-oak-tiktok.mp4
2024-oak-yt-short.mp4
2024-oak-yt.mp4


This naming scheme drives the entire automation pipeline.

High-Level Architecture
flowchart TD

A[Form Submission\n(Property Data)] --> B[NocoDB: Create Row]
B --> C[AI: Generate Platform Copy]
C --> D[NocoDB: Update Row w/ Text]

D --> E[Google Drive: Search Folder]
E --> F[Code: Parse Filenames]
F --> G[Download Videos]

G --> H[Switch on platformTag]

H --> H1[FB Portrait] --> U1[Upload to Blotato] --> W1[Write URL to Noco] --> X1[Get Fields] --> Y1[Publish Facebook Post] --> Z1[Save FB Post ID]

H --> H2[YT Short] --> U2[Upload Video] --> W2[Write URL] --> X2[Get Fields] --> Y2[Publish YT Short] --> Z2[Save Post ID]

H --> H3[YT Full] --> U3[Upload Video] --> W3[Write URL] --> X3[Get Fields] --> Y3[Publish YT Full] --> Z3[Save Post ID]

Trigger: Listing Information Form
On form submission

A simple n8n form where the user enters:

Property Name

Address

Bedrooms

Bathrooms

Sqft

Rent

Marketing Title

Marketing Description

Output is fed directly into:

Create Initial Row (NocoDB)

Creates a new listing entry in your NocoDB “social posts” table.
Primary key returned = Id.

AI: Generate Platform-Specific Text
Nodes:

OpenAI Chat Model4

Structured Output Parser

AI Generate Post Text

The prompt produces strict JSON with five sections:

tiktok

fbReel

fbPost

ytShort

ytFull

You enforce Blaze Voice guidelines and strict formatting.

Update a row (NocoDB)

Saves all generated text for later reuse in the publish stage.

Locate Videos in Google Drive
SearchDrive

Fetches all files in /social-media-videos/.

Parse File Names (Code Node)

Groups files by slug and assigns:

Suffix	platformTag
-tiktok	tiktok
-fb-reel	fb-reel
-fb-prt	fb-prt
-yt-short	yt-short
-yt	yt

Outputs one item per video, each containing:

platformTag

videoFileId

txtFileId

slug

Drive Download Videos

Downloads the binary video blob into binary.data.

Switch: Route by Platform
Switch

Routes each video to the correct branch:

platformTag	Branch Node
fb-prt	Fb Portrait Upload
tiktok	(future expansion)
fb-reel	(future expansion)
yt-short	YT Short Upload
yt	YT Full Upload
Upload + Publish (Blotato)
FB Portrait Branch

Fb Portrait Upload

Update Facebook Reel URL (save video URL to NocoDB)

Get FB Reel Post Fields

Create post2 (Facebook publish)

Update a row2 (save FB post ID)

YouTube Short Branch

YT Short Upload

Update YT Short Vid URL

Get YT Short Fields

Create post

Update a row4 (YT Short ID)

YouTube Full Branch

YT Full Upload

Update Youtube URL

Get YT Full Fields

Create post3

Update a row6 (YT Full ID)

All three branches store:

Video URL

Post ID
back into your NocoDB record.

Data Stored in NocoDB

Each listing row ends up containing:

Original listing data

AI-generated text for each platform

Video URLs (FB, YT Short, YT Full)

Post IDs for each platform

This gives you full auditability and lets you generate reporting later (views, CTR, etc.).

Manual Steps Required

Even though the automation is now operational, you still must do two things:

1. Create the listing record

Submit the n8n form with all property info.

2. Export the videos properly

Manually create:

Vertical + horizontal versions

Saved in Google Drive with the exact suffix structure defined above

Once the files hit Drive, the rest is hands-off.

Upgrade Paths (Recommended)

Here are ways to take friction from “manual + automated” to fully automated.

1. Auto-Generate Videos (High ROI)

Use:

Veo 3.1

D-ID

OpenAI Video

Runway

Adobe API

Pipeline:

Detect new listing added to NocoDB.

Feed data into your video template.

Auto-render FB, TikTok, YT versions.

Save into Drive automatically.

Trigger your exact publishing workflow.

Outcome: Zero manual editing unless needed.

2. Auto-Generate the Listing Information

Scrape or derive from:

AppFolio

Your website listing

A JSON spec in Google Drive

A simple “Describe the property” text box

Images alone (AI description + structured data extraction)

This removes the form entry part.

3. Auto-Publish to TikTok + IG + X (Future)

Blotato or another unified API will eventually support these fully.

Pipeline upgrade:

Add three nodes off the Switch:

TikTok Upload + Post

IG Reels Upload + Post

X Video Upload + Post

4. Add Slack/Discord/Email Notifications

As soon as a post publishes:

“2024 Oak is now live on Facebook/YT”

Include media links + post IDs

Automatically send to #marketing or #leasing

5. Error Recovery + Retry System

Because uploads sometimes fail, build:

Catch node

Logging table in NocoDB

Retry queue

“Try again” button

6. Multi-Property Batch Mode

Instead of running one listing at a time:

Watch the Drive folder for any new .txt file

Auto-detect all slugs

Process in batches

Notify when complete

Efficiency Improvements Inside the Workflow
✔ Remove redundant “Get Fields” queries

You can merge Noco fields forward instead of re-fetching them for each branch.

✔ Switch earlier

Move AI → Noco → Drive steps in parallel per platform.

✔ Refactor Noco updates

One update at the end instead of multiple incremental updates.

✔ Keep binary + metadata together using Merge Nodes

Protects against data loss if you add more lookups later.
