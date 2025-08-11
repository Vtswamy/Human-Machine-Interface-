# ===============================
# requirements.txt
# ===============================
opencv-python
mediapipe
pyautogui

# ===============================
# Main.py
# ===============================
import cv2
import mediapipe as mp
import pyautogui

# Initialize webcam
cap = cv2.VideoCapture(0)

# Mediapipe hand tracking setup
mp_hands = mp.solutions.hands
hands = mp_hands.Hands(max_num_hands=1)  # Track only one hand
mp_draw = mp.solutions.drawing_utils

# Get screen size
screen_width, screen_height = pyautogui.size()

# Main loop
while True:
    success, img = cap.read()
    if not success:
        break

    img = cv2.flip(img, 1)  # Mirror image for natural control
    img_rgb = cv2.cvtColor(img, cv2.COLOR_BGR2RGB)
    results = hands.process(img_rgb)
    img_height, img_width, _ = img.shape

    if results.multi_hand_landmarks:
        for hand_landmarks in results.multi_hand_landmarks:
            lm_list = []
            for id, lm in enumerate(hand_landmarks.landmark):
                x, y = int(lm.x * img_width), int(lm.y * img_height)
                lm_list.append((id, x, y))
                if id == 4 or id == 8:  # Thumb tip & index tip
                    cv2.circle(img, (x, y), 10, (0, 255, 255), cv2.FILLED)

            mp_draw.draw_landmarks(img, hand_landmarks, mp_hands.HAND_CONNECTIONS)

            # Move mouse with index fingertip
            x_index, y_index = lm_list[8][1], lm_list[8][2]
            mouse_x = int(screen_width / img_width * x_index)
            mouse_y = int(screen_height / img_height * y_index)
            pyautogui.moveTo(mouse_x, mouse_y)

            # Click if index and thumb are close
            x_thumb, y_thumb = lm_list[4][1], lm_list[4][2]
            distance = ((x_thumb - x_index) ** 2 + (y_thumb - y_index) ** 2) ** 0.5
            if distance < 20:
                pyautogui.click()

    cv2.imshow("Hand Gesture Mouse Control", img)
    if cv2.waitKey(1) & 0xFF == 27:  # ESC key
        break

cap.release()
cv2.destroyAllWindows()

# ===============================
# README.md
# ===============================
# Hand Gesture Mouse Control (Python + OpenCV + Mediapipe)

This project lets you control your computer mouse using **hand gestures** detected via your webcam.  
It uses **OpenCV** for video input, **Mediapipe** for landmark detection, and **PyAutoGUI** for mouse control.

---

opencv-python
mediapipe
pyautogui## 📦 Requirements
Create a `requirements.txt` file with:
opencv-python
mediapipe
pyautogui


---

## ⚙ Installation
Make sure Python 3.7+ is installed.


Clone or download
git clone <your_repo_url>
cd <your_project_folder>

Install dependencies
pip install -r requirements.txt


---

## ▶ Usage
Run:
python Main.py


---

## 🖐 Gestures
- **Move Mouse** → Move your **index finger tip** (landmark 8) in the air.
- **Click** → Bring **thumb tip** (landmark 4) close to **index finger tip**.
- **Exit** → Press `ESC`.

---

## 💡 Tips
- Ensure good lighting and avoid background clutter.
- Keep hand visible to camera.
- Move slowly at first to get used to tracking sensitivity.
