# Project Context: Developer Tools

## Project Overview
This directory contains a collection of standalone, client-side tools designed to run directly in a web browser. The primary focus is on simplicity, portability, and zero dependencies.

The current flagship tool is **Text Transformations**, which allows users to perform common string manipulations (case conversion, JSON formatting) locally.

## Architecture & Technology
*   **Type:** Static Web Site / Collection of HTML Utilities.
*   **Stack:** Vanilla HTML5, CSS3, and JavaScript (ES6+).
*   **Structure:** Single-file components. Styles and scripts are embedded directly within the HTML files to ensure they are self-contained and easy to share or deploy without a build step.
*   **Styling:** Custom CSS with CSS Variables for theming. Includes automatic Light/Dark mode support via `@media (prefers-color-scheme: dark)`.

## Key Files
*   **`text-transformations/text-transformations.html`**: The main utility file. Contains the UI and logic for:
    *   Uppercase/Lowercase conversion.
    *   Title Case conversion.
    *   JSON Prettifying/Validation.
*   **`README.md`**: Serves as the entry point/index for the collection.

## Building and Running
Since this project uses vanilla web technologies without a build system:
1.  **No Build Step:** There is no `npm install` or compilation required.
2.  **Run:** Simply open the desired `.html` file (e.g., `text-transformations/text-transformations.html`) in any modern web browser.
    *   From terminal: `open text-transformations/text-transformations.html` (macOS) or `start text-transformations/text-transformations.html` (Windows).

## Development Conventions
*   **Single File Policy:** Keep HTML, CSS, and JS in the same file to maintain portability unless the tool becomes significantly complex.
*   **Zero Dependencies:** Avoid external libraries (like React, jQuery, or Bootstrap) unless absolutely necessary.
*   **Theming:** All tools should respect the user's system color scheme preferences (Dark/Light mode).
*   **Simplicity:** Code should be readable and standard-compliant.
