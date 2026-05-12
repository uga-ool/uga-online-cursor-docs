# Peer Assessment of Group Work Brightspace App

This tool is intended to help collect feedback about students' experience working on group projects.

## Technical Details

This app is built with ReactJS using Vite build tools. It runs completely within the Brightspace Learning Management System using the context of the logged in user.

## How to Deploy

1. Download the repo
1. `npm install`
1. Rename `vite.sample.config.ts` to `vite.config.ts`
1. Edit the base url in `vite.config.ts` to match your intended upload location. It will be something like `/shared/peerassessment`
1. `npm run build` will generate both the dist folder and the dist_zip folder
1. Upload `dist.zip` from the dist_zip folder to the Public Files are in Brightspace. The upload location should correspond to the base URL set above.
1. Unzip `dist.zip` inside Public Files.
1. Add index.html to any course you want to use the tool. You may rename index.html with no issues.

## How to Use

See the [instructions document](https://docs.google.com/document/d/1DoEO4fk99XX7jlfm0G_HtF80F9ewqciHnE6Eg9pe7dA/edit#heading=h.rl38p3u3ww2j).