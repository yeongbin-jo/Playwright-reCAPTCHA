[![Python](https://img.shields.io/badge/Python-3.8+-blue.svg)](https://www.python.org/downloads/release/python-380/)
[![PyPI](https://img.shields.io/pypi/v/playwright-recaptcha.svg)](https://pypi.org/project/playwright-recaptcha/)
[![Downloads](https://img.shields.io/pypi/dm/playwright-recaptcha.svg)](https://pypi.org/project/playwright-recaptcha/)
[![License](https://img.shields.io/badge/license-MIT-green)](https://github.com/Xewdy444/Playwright-reCAPTCHA/blob/main/LICENSE)
[![Code Style](https://img.shields.io/badge/code%20style-black-000000.svg)](https://github.com/psf/black)

# Playwright-reCAPTCHA
A Python library for solving reCAPTCHA v2 and v3 with Playwright.

## Solving reCAPTCHA v2
reCAPTCHA v2 is solved by transcribing the audio challenge using the Google speech recognition API and entering the text as the response.

## Solving reCAPTCHA v3
The solving of reCAPTCHA v3 is done by the browser itself, so this library simply waits for the browser to make a POST request to https://www.google.com/recaptcha/api2/reload or https://www.google.com/recaptcha/enterprise/reload and parses the response to get the `g-recaptcha-response` token.

---

All solvers return the `g-recaptcha-response` token, which is required for form submissions.

It's important to note that reCAPTCHA v3 uses a token-based scoring system. Each user's token is automatically assigned a score based on their interactions with the website. The score is used to determine the likelihood of the user being a human or a bot. The token is then passed to the web server, and the website owner decides what action to take based on the score.

# Installation
```
pip install playwright-recaptcha
```

This library requires FFmpeg to be installed on your system in order to convert the audio challenge from reCAPTCHA v2 into text.

|   OS    |           Command           |
| :-----: | :-------------------------: |
| Debian  | sudo apt-get install ffmpeg |
|  MacOS  |     brew install ffmpeg     |
| Windows |    choco install ffmpeg     |

You can also download the latest static build from [here](https://ffmpeg.org/download.html).

> **Note**
> Make sure to have the ffmpeg and ffprobe binaries in your system's PATH so that the SpeechRecognition library can find them.

# Examples

## reCAPTCHA v2

### Synchronous
```py
from playwright.sync_api import sync_playwright
from playwright_recaptcha import recaptchav2

with sync_playwright() as playwright:
    browser = playwright.firefox.launch()
    page = browser.new_page()

    page.goto(
        "https://www.google.com/recaptcha/api2/demo", wait_until="networkidle"
    )

    with recaptchav2.SyncSolver(page) as solver:
        token = solver.solve_recaptcha()
        print(token)
```

### Asynchronous
```py
import asyncio
from playwright.async_api import async_playwright
from playwright_recaptcha import recaptchav2

async def main() -> None:
    async with async_playwright() as playwright:
        browser = await playwright.firefox.launch()
        page = await browser.new_page()

        await page.goto(
            "https://www.google.com/recaptcha/api2/demo", wait_until="networkidle"
        )

        async with recaptchav2.AsyncSolver(page) as solver:
            token = await solver.solve_recaptcha()
            print(token)

asyncio.run(main())
```

## reCAPTCHA v3

### Synchronous
```py
from playwright.sync_api import sync_playwright
from playwright_recaptcha import recaptchav3

with sync_playwright() as playwright:
    browser = playwright.firefox.launch()
    page = browser.new_page()
    page.goto("https://antcpt.com/score_detector/")

    with recaptchav3.SyncSolver(page) as solver:
        token = solver.solve_recaptcha()
        print(token)
```

### Asynchronous
```py
import asyncio
from playwright.async_api import async_playwright
from playwright_recaptcha import recaptchav3

async def main() -> None:
    async with async_playwright() as playwright:
        browser = await playwright.firefox.launch()
        page = await browser.new_page()
        await page.goto("https://antcpt.com/score_detector/")

        async with recaptchav3.AsyncSolver(page) as solver:
            token = await solver.solve_recaptcha()
            print(token)

asyncio.run(main())
```

# Exceptions
|        Exception        |                                                                      Description                                                                      |
| :---------------------: | :---------------------------------------------------------------------------------------------------------------------------------------------------: |
|     RecaptchaError      |                            The base class for reCAPTCHA exceptions, used as a catch-all for any reCAPTCHA-related errors.                             |
|   RecaptchaSolveError   |                An exception raised when the reCAPTCHA could not be solved, used as a catch-all for any reCAPTCHA solve-related errors.                |
| RecaptchaNotFoundError  |                                        An exception raised when the reCAPTCHA v2 was not found on the website.                                        |
| RecaptchaRateLimitError | An exception raised when the reCAPTCHA rate limit has been exceeded. This can happen if the library is being used to solve reCAPTCHA v2s too quickly. |
|  RecaptchaTimeoutError  |                                An exception raised when the reCAPTCHA v3 was not solved within the specified timeout.                                 |

# Disclaimer
This library is intended for use in automated testing and development environments only and should not be used for any illegal or malicious purposes. Any use of this library for activities that violate the terms of service of any website or service is strictly prohibited. The contributors of this library will not be held liable for any damages or legal issues that may arise from the use of this library. By using this library, you agree to these terms and take full responsibility for your actions.
