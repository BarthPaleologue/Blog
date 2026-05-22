---
title: "End-to-end testing for web games"
author: "Barthélemy Paléologue"
type: ""
date: 2025-04-30T23:14:01+02:00
subtitle: "A minimal setup that just works"
image: ""
tags: ["miscellaneous", "cosmos-journeyer", "webgpu"]
---

Hello everyone!

It's been a while since I last posted. I was busy getting a job among other things, but now I'm back, and I have been cooking!

## TLDR

- Minimal setup on [GitHub](https://github.com/BarthPaleologue/BabylonPlaywrightExample)
- Simple [Dockerfile](https://github.com/BarthPaleologue/BabylonPlaywrightExample/blob/master/Dockerfile) to run the tests
- [GitHub Actions CI](https://github.com/BarthPaleologue/BabylonPlaywrightExample/blob/master/.github/workflows/e2e.yml) to run the tests on PRs

## Introduction

In the past few months, my space exploration game, [Cosmos Journeyer](https://cosmosjourneyer.com), has been growing quite a lot, creating new challenges that some of you are probably familiar with if you are making any large project.

Today's problem can be summed up with this chart:

<br>

{{<figure src="./image.png" alt="New features goes down, testing goes up" caption="I guess we are doing tests now" caption-position="bottom" caption-effect="fade">}}

<br>

As you can see, the more features gets added, the more time it takes to test all of them. This means less time spent adding new features.

This slows the pace of development, which can lead to loss of motivation, and ultimately, the death of the project.

While unit tests are great for testing the inner logic of your game, they are not very useful for ensuring the right thing is displayed on the screen, where a lot of things can go wrong. This is where end-to-end tests come in: testing the game, like a player would do, but automatically.

Today, I will show you how to set up a simple end-to-end testing system for web games using [Playwright](https://playwright.dev/) that will allow you to test your web game in a browser, and take screenshots to ensure that the right thing is displayed on the screen.

## Prerequisites

This tutorial assumes you are using [Node.js](https://nodejs.org/en/) to develop your game. If you are using another runtime, Playwright might work differently, or not at all.

If you don't have a project yet, you can fork [this template](https://github.com/eldinor/bp800) that provides a minimal setup for Babylon.JS to get you started.

## Setting up playwright

### Dependencies

The first step is to add playwright to our project. Using npm, this is as simple as running:

```bash
npm install --save-dev playwright
```

This will install playwright and its dependencies. Playwright needs some browsers to work with, they can be installed with the following command:

```bash
npx playwright install
```

This will not work on some systems, but don't worry, that's why there is a "Docker" section later.

### Configuration

Now we can create a new `playwright.config.ts` file in the root of our project:

```typescript
import { defineConfig } from "@playwright/test";

export default defineConfig({
    testDir: "tests/e2e", // directory where the tests are located
    retries: process.env.CI ? 2 : 0, // retry failed tests on CI
    use: {
        baseURL: "http://localhost:8080", // base URL where the game is served
        browserName: "chromium", // We are testing on chromium
        headless: true, // do not show the browser window
        launchOptions: {
            args: [
                "--no-sandbox",
                "--disable-dev-shm-usage",
                "--use-gl=swiftshader" // software WebGL (no need for a GPU)
            ]
        },
        viewport: { width: 1280, height: 720 } // HD resolution is good enough
    },

    expect: {
        toHaveScreenshot: {
            maxDiffPixelRatio: 0.03, // ~3 % pixels may differ (aliasing can change the color of some pixels)
            threshold: 0.01 // pixels are considered different if they differ by more than 1% (aliasing can change the color of some pixels)
        }
    },

    webServer: {
        command: "npm run serve:prod", // Change this to the command that serves your built game
        url: "http://localhost:8080", // URL where the game is served
        reuseExistingServer: !process.env.CI
    }
});
```

This config will use a Chromium browser running in HD for the tests. Some fields need to be changed depending on your own project.

Let's write some tests now!

## Writing a simple test

Let's create a new file in the `tests/e2e` directory called `firstFrame.spec.ts` which will take a screenshot of the first frame of the game and compare it to a baseline image.

```typescript
import { expect, test } from "@playwright/test";

test("first frame should render correctly", async ({ page }) => {
    await page.goto(`/`); // load the game at the root URL

    await page.locator('#renderCanvas[data-ready="1"]').waitFor({ timeout: 15_000 }); // wait for the canvas to have a data-ready attribute set to 1

    await expect(page.locator("#renderCanvas")).toHaveScreenshot(`firstFrame.png`, { timeout: 15_000 }); // take a screenshot of the canvas and compare it to the baseline image
});
```

As you can see, playwright allows us to wait for a custom flag to be set on the canvas element before taking the screenshot. This is useful to ensure that the game is fully loaded before taking the screenshot. To set this flag, simply add the following line to your game code after the first frame is rendered:

```typescript
canvas.dataset.ready = "1";
```

In this test, my canvas has an id of `renderCanvas`, don't forget to change it to the id of your own canvas element.

This is a basic test, but that already goes a long way to test shader outputs and catch visual regressions.

If you want deeper tests, you can look into adding your game logic to the `window` object so that you can change the game state from the test and take multiple screenshots! Playwright also allows you to interact with the DOM, so you can click on buttons, press keys, etc.

## Running the tests

Let's get this running. This section assumes you could install playwright's browsers earlier. Skip to the next section otherwise.

First, let's generate the baseline images:

```bash
npx playwright test --update-snapshots
```

This should create a new folder in the `tests/e2e` directory with the new baseline image. Each test file will have its own folder. 

Then, you can run the tests with:

```bash
npx playwright test
```

Hopefully, all the tests should pass. Try changing something in your game and run the tests again. You should have a failure with a diff image showing the differences between the baseline image and the new image.

## Make it run in a docker

The issue with Playwright is that you can't run it on all systems. That can be an issue for an open source project where contributors are using different systems, and the CI is using yet another system. That's why Docker exists!

<br>

{{<figure src="./image-1.png" alt="Docker creation meme" caption="Mom said it's my turn to reuse this meme" caption-position="bottom" caption-effect="fade">}}

<br>

Here is a minimal `Dockerfile` for running the tests:

```dockerfile
ARG PW_VERSION=1.52.0 # Change this to the version you want to use
FROM mcr.microsoft.com/playwright:v${PW_VERSION}-jammy AS e2e # official playwright image

WORKDIR /app
COPY package.json package-lock.json ./ # copy package.json and package-lock.json to the image
RUN npm ci && npm i -D @playwright/test@${PW_VERSION} # install dependencies and the correct version of playwright

COPY . . # copy all other sources
RUN npm run build # build the game

# run the tests
CMD ["npx","playwright","test","--reporter=line"]
```

You will also need a `.dockerignore` file to only copy the sources of your project:

```
node_modules
dist <-- or whatever your build folder is
```

Now you can run the tests on any system that supports Docker with this command:

```bash
docker build \
-t game-e2e \
--build-arg PW_VERSION=1.52.0 . \
&& docker run --rm --ipc=host -e CI=1 \
-v \"$(pwd)/tests/e2e:/app/tests/e2e\" \ 
-v \"$(pwd)/artifacts:/output\" game-e2e \
npx playwright test --output=/output --reporter=line,html
```

If you don't know Docker, this can look scary, so here is a commented version of each step:

```bash
docker build \ # build a docker image
-t game-e2e \ # with the name game-e2e
--build-arg PW_VERSION=1.52.0 . \ # using a specific version of playwright
&& docker run --rm --ipc=host -e CI=1 \ # run the image with the --rm flag to remove it after use
-v \"$(pwd)/tests/e2e:/app/tests/e2e\" \ # mount the tests folder to the image so that baseline images are saved
-v \"$(pwd)/artifacts:/output\" game-e2e \ # mount the artifacts folder to the image so that reports are saved
npx playwright test --output=/output --reporter=line,html # run the tests with the --output flag to save the reports in the artifacts folder
```

The first run will fail and generate the baseline images. The second run should pass. You can also run the tests with the `--update-snapshots` flag to update the baseline images.

Like before, try changing something in your game and run the tests again. You should have a failure with a diff image showing the differences between the baseline image and the new image.

## Running the tests on PRs

Now, checking you didn't break anything is all well and good, but what about the other contributors? You probably also want to check their PRs as well!

For that, we can run a CI check leveraging our newly written Dockerfile to run the tests in a GitHub Action workflow `e2e.yml`:

```yaml
name: e2e
on:
    push:
        branches: [main]
    pull_request:
        branches: [main]
    workflow_dispatch:

jobs:
    playwright-e2e:
        runs-on: ubuntu-latest

        steps:
            # 1 ─────────── repo + LFS
            - uses: actions/checkout@v4

            # 2 ─────────── build test image (local load, no push)
            - name: Build Docker image
              uses: docker/build-push-action@v5
              with:
                  context: .
                  tags: app-e2e:${{ github.sha }}
                  load: true # load into the runner’s daemon
                  build-args: |
                      PW_VERSION=1.52.0

            # 3 ─────────── run Playwright
            - name: Run Playwright suite
              run: |
                  mkdir -p artifacts                 # host dir for reports / diffs
                  docker run --rm --ipc=host -e CI=1 \
                    -v ${{ github.workspace }}/tests/e2e:/app/tests/e2e \
                    -v ${{ github.workspace }}/artifacts:/output \
                    app-e2e:${{ github.sha }} \
                    npx playwright test \
                        --output=/output \
                        --reporter=line,html

            # 4 ─────────── publish results
            - name: Upload HTML report
              if: always() # even on failure
              uses: actions/upload-artifact@v4
              with:
                  name: playwright-report
                  path: artifacts/playwright-report

            - name: Upload raw diffs (only on failure)
              if: failure()
              uses: actions/upload-artifact@v4
              with:
                  name: test-results
                  path: artifacts/test-results
```

Add this file to the `.github/workflows` folder of your project. The end-to-end tests will now run on every push to the `main` branch and on every pull request. The results will be uploaded as artifacts that you can download from the GitHub Actions page.

## Conclusion

I hope you found this tutorial useful! Let me know if you have any questions or issues with the setup. I will be happy to help you out.

All the code for this tutorial is available on [GitHub](https://github.com/BarthPaleologue/BabylonPlaywrightExample) under the MIT license. It contains a small Babylon.JS scene used to generate the screenshots, and the Dockerfile from this post, as well as the GitHub Actions script.

Until next time, have a nice day!
