import threading
import cv2
from deepface import DeepFace

# Initialize video capture
cap = cv2.VideoCapture(0, cv2.CAP_DSHOW)
cap.set(cv2.CAP_PROP_FRAME_WIDTH, 640)
cap.set(cv2.CAP_PROP_FRAME_HEIGHT, 480)

counter = 0
face_match = False
reference_img = cv2.imread("Meneesh.jpg")
reference_img = cv2.imread("Hanish.jpg")
reference_img = cv2.imread("Rupesh.jpg")

# Thread-safe variable to hold face_match status
face_match_event = threading.Event()

def checkface(frame):
    global face_match
    try:
        # Verify if the face in the current frame matches the reference image
        if DeepFace.verify(frame, reference_img.copy())['verified']:
            face_match = True
        else:
            face_match = False
        face_match_event.set()  # Signal that the face_match status is updated
    except ValueError:
        face_match = False
        face_match_event.set()  # Signal that the face_match status is updated

while True:
    ret, frame = cap.read()

    if ret:
        if counter % 30 == 0:  # Run face matching check every 30 frames
            try:
                threading.Thread(target=checkface, args=(frame.copy(),)).start()
            except ValueError:
                pass
        counter += 1

        # Wait until the face match status is updated
        face_match_event.wait()
        face_match_event.clear()  # Reset event for the next check

        # Display the result on the frame
        if face_match:
            cv2.putText(frame, "Match!", (20, 450), cv2.FONT_HERSHEY_COMPLEX_SMALL, 2, (0, 255, 0), 3)
        else:
            cv2.putText(frame, "No match!", (20, 450), cv2.FONT_HERSHEY_COMPLEX_SMALL, 2, (0, 0, 250), 3)

        # Show the frame with the corresponding text
        cv2.imshow("Video", frame)

        # Wait for keypress to exit (1 ms to keep the video loop responsive)
        key = cv2.waitKey(1)
        if key == ord("Q"):  # If the "Q" key is pressed, exit the loop
            print("Scam")
            break

# Release the video capture object and close all windows
cap.release()
cv2.destroyAllWindows()
