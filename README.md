# Air-canvas
This project is about a canvas to draw on the screen by using hand gestures


# All the imports go here 
import cv2 
import numpy as np 
import mediapipe as mp 
from collections import deque 
# Giving different arrays to handle colour points of different colour 
bpoints = [deque(maxlen=1024)] 
gpoints = [deque(maxlen=1024)] 
rpoints = [deque(maxlen=1024)] 
ypoints = [deque(maxlen=1024)] 
# These indexes will be used to mark the points in particular arrays of specific colour 
blue_index = 0 
green_index = 0 
red_index = 0 
yellow_index = 0 
#The kernel to be used for dilation purpose  
kernel = np.ones((5,5),np.uint8) 
colors = [(255, 0, 0), (0, 255, 0), (0, 0, 255), (0, 255, 255)] 
colorIndex = 0 
action_stack = [] 
# Here is code for Canvas setup 
paintWindow = np.zeros((480,736,3)) + 255 
cv2.namedWindow('Paint', cv2.WINDOW_AUTOSIZE) 
# initialize mediapipe 
mpHands = mp.solutions.hands 
hands = mpHands.Hands(max_num_hands=1, min_detection_confidence=0.7) 
mpDraw = mp.solutions.drawing_utils 
# Initialize the webcam 
cap = cv2.VideoCapture(0) 
ret = True 
while ret: 
# Read each frame from the webcam 
ret, frame = cap.read() 
x, y, c = frame.shape 
# Flip the frame vertically 
frame = cv2.flip(frame, 1)  
#hsv = cv2.cvtColor(frame, cv2.COLOR_BGR2HSV) 
framergb = cv2.cvtColor(frame, cv2.COLOR_BGR2RGB) 
frame = cv2.rectangle(frame, (10,1), (100,65), (0,0,0), 2) 
frame = cv2.rectangle(frame, (120,1), (200,65), (255,0,0), 2) 
frame = cv2.rectangle(frame, (210,1), (290,65), (0,255,0), 2) 
frame = cv2.rectangle(frame, (300,1), (395,65), (0,0,255), 2) 
frame = cv2.rectangle(frame, (405,1), (500,65), (0,255,255), 2) 
frame = cv2.rectangle(frame, (510,1), (600,65), (0,0,0), 2) 
cv2.putText(frame, "CLEAR", (20, 33), cv2.FONT_HERSHEY_SIMPLEX, 0.5, (0, 0, 0), 2, 
cv2.LINE_AA) 
cv2.putText(frame, "BLUE", (130, 33), cv2.FONT_HERSHEY_SIMPLEX, 0.5, (0, 0, 0), 2, 
cv2.LINE_AA) 
cv2.putText(frame, "GREEN", (220, 33), cv2.FONT_HERSHEY_SIMPLEX, 0.5, (0, 0, 0), 
2, cv2.LINE_AA) 
cv2.putText(frame, "RED", (310, 33), cv2.FONT_HERSHEY_SIMPLEX, 0.5, (0, 0, 0), 2, 
cv2.LINE_AA) 
cv2.putText(frame, "YELLOW", (415, 33), cv2.FONT_HERSHEY_SIMPLEX, 0.5, (0, 0, 
0), 2, cv2.LINE_AA) 
cv2.putText(frame, "UNDO", (520, 33), cv2.FONT_HERSHEY_SIMPLEX, 0.5, (0, 0, 0), 
2, cv2.LINE_AA) 
#frame = cv2.cvtColor(hsv, cv2.COLOR_HSV2BGR) 
# Get hand landmark prediction 
result = hands.process(framergb) 
# post process the result. 
if result.multi_hand_landmarks: 
landmarks = [] 
for handslms in result.multi_hand_landmarks: 
for lm in handslms.landmark: 
# # print(id, lm) 
# print(lm.x) 
# print(lm.y) 
lmx = int(lm.x * 640) 
lmy = int(lm.y * 480) 
landmarks.append([lmx, lmy]) 
# Drawing landmarks on frames 
mpDraw.draw_landmarks(frame, handslms, mpHands.HAND_CONNECTIONS) 
fore_finger = (landmarks[8][0],landmarks[8][1])       
center = fore_finger 
thumb = (landmarks[4][0],landmarks[4][1]) 
cv2.circle(frame, center, 3, (0,255,0),-1) 
if thumb[1] - center[1] < 30:  # Pinch gesture 
# Push current state to stack before clearing 
action_stack.append((list(bpoints), list(gpoints), list(rpoints), list(ypoints))) 
elif center[1] <= 65: 
if 40 <= center[0] <= 140:  # Clear Button 
# Save the current state to history before clearing 
action_stack.append((list(bpoints), list(gpoints), list(rpoints), list(ypoints))) 
# Clear the points 
bpoints = [deque(maxlen=512)] 
gpoints = [deque(maxlen=512)] 
rpoints = [deque(maxlen=512)] 
ypoints = [deque(maxlen=512)] 
blue_index = 0 
green_index = 0 
red_index = 0 
yellow_index = 0 
# Clear the canvas 
paintWindow[:] = 255  # Set the entire canvas to white 
elif 120 <= center[0] <= 200: 
colorIndex = 0  # Blue 
elif 210 <= center[0] <= 290: 
colorIndex = 1  # Green 
elif 300 <= center[0] <= 395: 
colorIndex = 2  # Red 
elif 405 <= center[0] <= 500:  
colorIndex = 3  # Yellow 
elif 510 <= center[0] <= 600:  # Undo Button 
if action_stack: 
bpoints, gpoints, rpoints, ypoints = action_stack.pop()  # Restore the last state 
else: 
# Draw on the canvas 
if colorIndex == 0: 
bpoints[blue_index].appendleft(center) 
elif colorIndex == 1: 
gpoints[green_index].appendleft(center) 
elif colorIndex == 2: 
rpoints[red_index].appendleft(center) 
elif colorIndex == 3: 
ypoints[yellow_index].appendleft(center) 
# Draw lines of all the colors on the canvas and frame 
points = [bpoints, gpoints, rpoints, ypoints] 
# Draw lines of all the colors on the canvas and frame 
points = [bpoints, gpoints, rpoints, ypoints] 
# for j in range(len(points[0])): 
#         for k in range(1, len(points[0][j])): 
#             if points[0][j][k - 1] is None or points[0][j][k] is None: 
#             cv2.line(paintWindow, points[0][j][k - 1], points[0][j][k], colors[0], 2) 
for i in range(len(points)): 
for j in range(len(points[i])): 
for k in range(1, len(points[i][j])): 
if points[i][j][k - 1] is None or points[i][j][k] is None: 
continue 
cv2.line(frame, points[i][j][k - 1], points[i][j][k], colors[i], 2) 
cv2.line(paintWindow, points[i][j][k - 1], points[i][j][k], colors[i], 2)  
frame_resized = cv2.resize(frame, (paintWindow.shape[1], paintWindow.shape[0])) 
cv2.imshow("Output", frame_resized)  
cv2.imshow("Paint", paintWindow) 
if cv2.waitKey(1) == ord('q'): 
break 
# release the webcam and destroy all active windows 
cap.release() 
cv2.destroyAllWindows()  
<link rel="preconnect" href="https://fonts.googleapis.com">  
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>  
<link rel="preconnect" href="https://fonts.googleapis.com">  
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>  
<link rel="preconnect" href="https://fonts.googleapis.com">
     <link 
rel="preconnect" href="https://fonts.gstatic.com" crossorigin>    
src="https://cdn.jsdelivr.net/npm/bootstrap@5.1.3/dist/js/bootstrap.bundle.min.js" 
integrity="sha384- 
ka7Sk0Gln4gmtz2MlQnikT1wXgYsOg+OMhuP+IlRH9sENBO0LRn5q+8nbTov4+1p" 
crossorigin="anonymous"></script>  
</body>  
</html> 
