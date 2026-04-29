# MF** MFTF!

!!! question "While we're fixing Magento... why not fix the tests too?"

---

## The Elephant in the Room

If you've ever waited 45 minutes for an MFTF suite to crawl through Selenium WebDriver, you know the pain. MFTF (Magento Functional Testing Framework) was ambitious when it launched. But in 2026, it's:

- **Slow** — Selenium-based, single-threaded, waiting for every page load
- **Flaky** — random timeouts, stale element references, session crashes
- **Heavy** — requires Java, Selenium Server, ChromeDriver, Allure reports
- **Painful to write** — XML-based test definitions, action groups, sections, data entities — all to click a button
- **Maintenance burden** — thousands of XML files across modules that break on every UI change

Meanwhile, the rest of the testing world moved on.

---

## :bulb: The Idea: Replace MFTF with Playwright

**What if we replaced MFTF entirely with [Playwright](https://playwright.dev/)?**

Modern, fast, reliable end-to-end testing that developers actually enjoy writing.

| | MFTF | Playwright |
|---|------|-----------|
| **Language** | XML action groups | JavaScript/TypeScript (or PHP) |
| **Speed** | Minutes per test | Seconds per test |
| **Parallelism** | Barely | Native, out of the box |
| **Browser engines** | Selenium + ChromeDriver | Chromium, Firefox, WebKit — built in |
| **Dependencies** | Java, Selenium Server, Allure | `npm install` |
| **Debugging** | Good luck | Trace viewer, video recording, screenshots |
| **Auto-waiting** | Manual wait hacks | Built-in smart waiting |
| **CI/CD** | Heavy Docker images | Lightweight, fast startup |
| **Developer experience** | XML hell | Actual code you can read |

---

## Standing on Giants: elgentos/magento2-playwright

The brilliant team at **[elgentos](https://www.elgentos.nl/)** already started this work. Their [magento2-playwright](https://github.com/elgentos/magento2-playwright) project provides a Playwright testing framework specifically designed for Magento 2.

[:fontawesome-brands-github: elgentos/magento2-playwright](https://github.com/elgentos/magento2-playwright){ .md-button .md-button--primary }

### What They've Built

- Playwright test framework tailored for Magento 2 stores
- Page object models for common Magento pages
- Fixture support for test data
- CI/CD integration examples
- A real, working alternative to MFTF

### The elgentos Team

- **GitHub:** [github.com/elgentos](https://github.com/elgentos)
- **LinkedIn:** [elgentos on LinkedIn](https://www.linkedin.com/company/elgentos/)
- **Website:** [elgentos.nl](https://www.elgentos.nl/)

---

## How This Fits Into GMAD

As we decouple modules and make them optional, we need tests that verify:

1. **Module X can be disabled** without breaking anything
2. **Module X works correctly** when enabled
3. **Other modules still work** regardless of Module X's state

MFTF tests are tightly coupled to specific UI elements and page structures — exactly the kind of coupling we're trying to eliminate. Playwright tests can be written to test **behavior**, not **UI implementation details**.

!!! tip "The GMAD testing vision"
    Every decoupled module gets a Playwright test suite that verifies:
    
    - :white_check_mark: Module disabled → `bin/magento setup:di:compile` passes
    - :white_check_mark: Module disabled → storefront works without it
    - :white_check_mark: Module enabled → feature works correctly
    - :white_check_mark: No regressions in core functionality

---

## Want to Help?

This is a natural extension of the GMAD project. If you're passionate about testing and tired of MFTF:

1. Check out [elgentos/magento2-playwright](https://github.com/elgentos/magento2-playwright)
2. Help us define the test strategy for decoupled modules
3. [Open an issue](https://github.com/qoliber/great-magento-architecture-decoupling/issues) with your ideas

**Let's not just decouple Magento's modules — let's decouple it from bad testing tools too.**
