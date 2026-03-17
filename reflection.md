# 💭 Reflection: Game Glitch Investigator

## 1. What was broken when you started?

When I first ran the game, it was completely unplayable. The very first thing I noticed was that the hints were backwards — when I entered 1, the game told me to go lower, and when I entered 100, it told me to go higher, which is the exact opposite of what should happen. The second bug I noticed right away was that the score was going negative — after a few wrong guesses, the final score showed something like -45, which made no sense for a guessing game. Beyond those two obvious issues, the New Game button also did not work properly — clicking it after a game over left the error message on screen and blocked any new input, so there was no way to start fresh without refreshing the page.

---

## 2. How did you use AI as a teammate?

I used GitHub Copilot (VS Code) as my main AI tool throughout this project. For each bug I found, I described the problem clearly and in plain language, then asked Copilot to find the root cause and apply a fix.

One example where Copilot was correct and helpful was the inverted hints bug. I described the issue simply — "when the guess is too high it says Go HIGHER, when it should say Go LOWER" — and Copilot immediately identified the exact lines in `check_guess()` and swapped the return messages correctly. I verified it by running the game and confirming the hints now pointed in the right direction.

One example where Copilot's suggestion was incomplete was the difficulty range bug. Copilot applied a fix that looked correct in the code, but when I actually played the game on Easy mode, the secret number was still 45 — which is outside the Easy range of 1–20. The fix had not accounted for the fact that `st.session_state` only initialises once per session, so switching difficulty never regenerated the secret. I had to go back to Copilot with more detail about the root cause before it applied the correct fix.

---

## 3. Debugging and testing your fixes

My main approach to deciding whether a bug was really fixed was to play the game myself after every change. I did not just trust Copilot's explanation of what it changed — I ran the app, tried the specific scenario that triggered the bug, and checked the result with my own eyes. For the difficulty range bug specifically, this is what caught the incomplete fix: the code looked fine but playing the game revealed the secret was still out of range.

For automated testing, I ran `pytest` against `tests/test_game_logic.py` after the refactor into `logic_utils.py`. The tests covered the core logic functions — `check_guess`, `parse_guess`, `update_score`, and `get_range_for_difficulty`. Initially pytest failed with a `ModuleNotFoundError` because it could not find `logic_utils`. After adding a `conftest.py` file with a path fix, all 3 tests passed. Seeing "3 passed in 0.01s" gave me confidence that the core logic was working correctly.

Copilot helped me understand why the `conftest.py` was needed — it explained that pytest runs tests from a different working directory context than the app itself, so the project root needed to be explicitly added to `sys.path`.

---

## 4. What did you learn about Streamlit and state?

Streamlit works differently from most frameworks because every time the user interacts with the app — clicking a button, entering text, changing a dropdown — the entire Python script reruns from top to bottom. This means any regular variable you create gets reset to its initial value on every interaction. To keep information alive between reruns (like the secret number, the score, or the attempt count), you have to store it in `st.session_state`, which is a dictionary that Streamlit preserves across reruns.

I would explain it to a friend like this: imagine every time you click a button, the whole app restarts like it was just opened — but `st.session_state` is like a notepad that Streamlit keeps on the table, so your variables can survive the restart. If you do not write something to that notepad, it disappears. The tricky part is that the notepad only gets set up once — so if you store something when the app first loads (like the secret number on Normal difficulty), it will still be there when you switch to Easy, because the setup code does not run again. That is exactly what caused Bug 5 in this project.

---

## 5. Looking ahead: your developer habits

One habit I want to carry forward from this project is always verifying fixes by running and testing the actual application, not just reading the code or trusting the AI's explanation. The difficulty range bug was a perfect example — the code looked correct after Copilot's first fix, but playing the game revealed the problem immediately. Code review alone would have missed it.

One thing I would do differently next time is to describe bugs more precisely from the start. When I gave Copilot vague descriptions, it sometimes produced surface-level fixes that missed the root cause. When I was more specific — including what the actual observed behaviour was, what the expected behaviour should be, and which function was likely involved — Copilot's fixes were much more accurate and complete on the first attempt.

This project changed how I think about AI-generated code because I went in expecting the bugs to be obvious and easy to spot, but several of them were subtle and only revealed themselves through actual use. AI can write code that looks clean and logical at a glance but still behaves incorrectly in ways that only show up at runtime — which means human testing and judgment will always be a necessary part of the development process, not an optional extra.
