import cv2
import mediapipe as mp
import random
import time

mp_hands = mp.solutions.hands
mp_drawing = mp.solutions.drawing_utils

hands = mp_hands.Hands(min_detection_confidence=0.7, min_tracking_confidence=0.7)

cap = cv2.VideoCapture(0)

# Game state variables
game_state = "waiting"  # waiting, countdown, show_result, game_over
countdown_start = 0
user_gesture = "UNKNOWN"
computer_gesture = ""
user_score = 0
computer_score = 0
round_number = 1
round_result = ""
game_winner = ""

def count_extended_fingers(hand_landmarks):
    """Count how many fingers are extended"""
    finger_tips = [8, 12, 16, 20]  # Index, Middle, Ring, Pinky
    finger_pips = [6, 10, 14, 18]
    
    extended_fingers = 0
    
    # Check each finger (except thumb)
    for tip, pip in zip(finger_tips, finger_pips):
        if hand_landmarks.landmark[tip].y < hand_landmarks.landmark[pip].y:
            extended_fingers += 1
    
    # Check thumb separately
    thumb_tip = hand_landmarks.landmark[4]
    thumb_ip = hand_landmarks.landmark[3]
    
    if abs(thumb_tip.x - thumb_ip.x) > 0.04:
        extended_fingers += 1
    
    return extended_fingers

def detect_gesture(hand_landmarks):
    """Detect Rock, Paper, or Scissors gesture"""
    extended = count_extended_fingers(hand_landmarks)
    
    if extended <= 1:
        return "ROCK"
    elif extended >= 4:
        return "PAPER"
    elif extended == 2 or extended == 3:
        index_tip = hand_landmarks.landmark[8]
        middle_tip = hand_landmarks.landmark[12]
        index_pip = hand_landmarks.landmark[6]
        middle_pip = hand_landmarks.landmark[10]
        
        index_extended = index_tip.y < index_pip.y
        middle_extended = middle_tip.y < middle_pip.y
        
        if index_extended and middle_extended:
            return "SCISSORS"
        else:
            return "UNKNOWN"
    else:
        return "UNKNOWN"

def determine_winner(user, computer):
    """Determine the winner of the round"""
    if user == computer:
        return "TIE"
    elif (user == "ROCK" and computer == "SCISSORS") or \
         (user == "PAPER" and computer == "ROCK") or \
         (user == "SCISSORS" and computer == "PAPER"):
        return "USER"
    else:
        return "COMPUTER"

def draw_text(frame, text, position, font_scale=1, color=(255, 255, 255), thickness=2):
    """Helper function to draw text with background"""
    font = cv2.FONT_HERSHEY_SIMPLEX
    text_size = cv2.getTextSize(text, font, font_scale, thickness)[0]
    x, y = position
    # Draw black background rectangle
    cv2.rectangle(frame, (x - 5, y - text_size[1] - 5), 
                  (x + text_size[0] + 5, y + 5), (0, 0, 0), -1)
    # Draw text
    cv2.putText(frame, text, position, font, font_scale, color, thickness)

while True:
    ret, frame = cap.read()
    if not ret:
        break
    
    frame = cv2.flip(frame, 1)
    h, w, _ = frame.shape
    
    rgb = cv2.cvtColor(frame, cv2.COLOR_BGR2RGB)
    results = hands.process(rgb)
    
    current_time = time.time()
    
    # Detect user gesture
    if results.multi_hand_landmarks:
        for hand_landmarks in results.multi_hand_landmarks:
            detected = detect_gesture(hand_landmarks)
            if detected != "UNKNOWN":
                user_gesture = detected
    
    # Game state machine
    if game_state == "waiting":
        draw_text(frame, f"Round {round_number}/3", (10, 50), 1.2, (0, 255, 255), 3)
        draw_text(frame, f"Score - You: {user_score} | Computer: {computer_score}", 
                  (10, 100), 1, (255, 255, 255), 2)
        draw_text(frame, "Show your gesture and press SPACE to start!", 
                  (10, h - 30), 0.8, (0, 255, 0), 2)
        draw_text(frame, f"Your gesture: {user_gesture}", (10, 150), 1, (255, 255, 0), 2)
        
        key = cv2.waitKey(1) & 0xFF
        if key == ord(' ') and user_gesture != "UNKNOWN":
            game_state = "countdown"
            countdown_start = current_time
            computer_gesture = random.choice(["ROCK", "PAPER", "SCISSORS"])
    
    elif game_state == "countdown":
        # Show result immediately
        game_state = "show_result"
        winner = determine_winner(user_gesture, computer_gesture)
        
        if winner == "USER":
            user_score += 1
            round_result = "You WIN this round!"
        elif winner == "COMPUTER":
            computer_score += 1
            round_result = "Computer WINS this round!"
        else:
            round_result = "It's a TIE! Play again!"
        
        countdown_start = current_time
    
    elif game_state == "show_result":
        draw_text(frame, f"Round {round_number}/3", (10, 50), 1.2, (0, 255, 255), 3)
        draw_text(frame, f"You: {user_gesture}", (10, 150), 1.2, (255, 255, 0), 3)
        draw_text(frame, f"Computer: {computer_gesture}", (10, 200), 1.2, (255, 0, 255), 3)
        draw_text(frame, round_result, (10, 280), 1.3, (0, 255, 0) if "WIN" in round_result else (255, 255, 255), 3)
        draw_text(frame, f"Score - You: {user_score} | Computer: {computer_score}", 
                  (10, 350), 1, (255, 255, 255), 2)
        
        if current_time - countdown_start > 3:
            if round_result != "It's a TIE! Play again!":
                # Only move to next round if there was no tie
                if round_number < 3:
                    round_number += 1
                    game_state = "waiting"
                    user_gesture = "UNKNOWN"
                else:
                    game_state = "game_over"
                    if user_score > computer_score:
                        game_winner = "YOU WIN THE GAME!"
                    elif computer_score > user_score:
                        game_winner = "COMPUTER WINS THE GAME!"
                    else:
                        game_winner = "IT'S A TIE GAME!"
            else:
                # If it's a tie, replay the same round
                game_state = "waiting"
                user_gesture = "UNKNOWN"
    
    elif game_state == "game_over":
        draw_text(frame, "GAME OVER", (w//2 - 150, 100), 2, (0, 255, 255), 4)
        draw_text(frame, f"Final Score - You: {user_score} | Computer: {computer_score}", 
                  (10, 200), 0.9, (255, 255, 255), 2)
        draw_text(frame, game_winner, (10, 280), 1.5, (0, 255, 0) if "YOU WIN" in game_winner else (255, 0, 0), 3)
        draw_text(frame, "Press 'R' to restart or 'Q' to quit", (10, h - 30), 0.8, (255, 255, 255), 2)
        
        key = cv2.waitKey(1) & 0xFF
        if key == ord('r'):
            # Reset game
            game_state = "waiting"
            user_score = 0
            computer_score = 0
            round_number = 1
            user_gesture = "UNKNOWN"
    
    cv2.imshow("Rock Paper Scissors Game", frame)
    
    if cv2.waitKey(1) & 0xFF == ord('q'):
        break

cap.release()
cv2.destroyAllWindows()
