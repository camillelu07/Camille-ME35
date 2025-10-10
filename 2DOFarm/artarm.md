from machine import Pin, PWM
import time

# Motor pins (Cytron Robo ESP32)
LEFT_MOTOR_PIN1 = Pin(27, Pin.OUT)
LEFT_MOTOR_PIN2 = Pin(14, Pin.OUT)
RIGHT_MOTOR_PIN1 = Pin(13, Pin.OUT)
RIGHT_MOTOR_PIN2 = Pin(12, Pin.OUT)

# Encoder pins (Cytron Robo ESP32)
# Grove 2 (Left Encoder) - uses pins 16 and 17
LEFT_ENCODER_A = Pin(16, Pin.IN, Pin.PULL_UP)
LEFT_ENCODER_B = Pin(17, Pin.IN, Pin.PULL_UP)
# Grove 3 (Right Encoder) - uses pins 25 and 32
RIGHT_ENCODER_A = Pin(25, Pin.IN, Pin.PULL_UP)
RIGHT_ENCODER_B = Pin(32, Pin.IN, Pin.PULL_UP)

# Servo pins
LOWER_SERVO = PWM(Pin(4), freq=50)
UPPER_SERVO = PWM(Pin(5), freq=50)

# Global encoder position counters
left_encoder_position = 0
right_encoder_position = 0

def set_servo_angle(servo, angle):
    """
    Set servo angle (0-180 degrees)
    """
    angle = max(0, min(180, angle))  # Constrain to 0-180
    # Convert angle to duty cycle (typically 40-115 for 0-180 degrees)
    min_duty = 40
    max_duty = 115
    duty = int(min_duty + (angle / 180) * (max_duty - min_duty))
    servo.duty(duty)

def left_encoder_callback(pin):
    """
    Read left encoder and update position
    """
    global left_encoder_position
    
    # Read both encoder channels
    a_state = LEFT_ENCODER_A.value()
    b_state = LEFT_ENCODER_B.value()
    
    # Determine direction (simple quadrature)
    # If A leads B, increment (clockwise), else decrement (counter-clockwise)
    if a_state == b_state:
        left_encoder_position += 1
    else:
        left_encoder_position -= 1
    
    # Map encoder position to 0-180 range
    servo_angle = int(left_encoder_position % 180)
    
    # Update servo
    set_servo_angle(LOWER_SERVO, servo_angle)
    
    print(f"Left Encoder: {left_encoder_position}, Servo Angle: {servo_angle}°")

def right_encoder_callback(pin):
    """
    Read right encoder and update position
    """
    global right_encoder_position
    
    # Read both encoder channels
    a_state = RIGHT_ENCODER_A.value()
    b_state = RIGHT_ENCODER_B.value()
    
    # Determine direction (simple quadrature)
    # If A leads B, increment (clockwise), else decrement (counter-clockwise)
    if a_state == b_state:
        right_encoder_position += 1
    else:
        right_encoder_position -= 1
    
    # Map encoder position to 0-180 range
    servo_angle = int(right_encoder_position % 180)
    
    # Update servo
    set_servo_angle(UPPER_SERVO, servo_angle)
    
    print(f"Right Encoder: {right_encoder_position}, Servo Angle: {servo_angle}°")

def setup():
    """
    Initialize system
    """
    print("=" * 50)
    print("DC Motor Encoder to Servo Mapper")
    print("Cytron Robo ESP32")
    print("=" * 50)
    
    # Stop motors (ensure they're off)
    LEFT_MOTOR_PIN1.off()
    LEFT_MOTOR_PIN2.off()
    RIGHT_MOTOR_PIN1.off()
    RIGHT_MOTOR_PIN2.off()
    
    # Initialize servos to 0 position
    set_servo_angle(LOWER_SERVO, 0)
    set_servo_angle(UPPER_SERVO, 0)
    print("Servos initialized to 0°")
    
    # Attach interrupts to encoder A channels
    LEFT_ENCODER_A.irq(trigger=Pin.IRQ_RISING | Pin.IRQ_FALLING, handler=left_encoder_callback)
    RIGHT_ENCODER_A.irq(trigger=Pin.IRQ_RISING | Pin.IRQ_FALLING, handler=right_encoder_callback)
    
    print("\nSetup complete!")
    print("Instructions:")
    print("1. Manually dial the LEFT motor → controls LOWER_SERVO (Pin 4)")
    print("2. Manually dial the RIGHT motor → controls UPPER_SERVO (Pin 5)")
    print("3. Encoder values are mapped 0-180 using modulo")
    print("4. Turn motors to move the arm servos!")
    print("\nPress Ctrl+C to exit\n")

def main():
    """
    Main loop
    """
    setup()
    
    try:
        while True:
            time.sleep(0.1)
    except KeyboardInterrupt:
        print("\n" + "=" * 50)
        print("Exiting program...")
        print(f"Final Left Encoder Position: {left_encoder_position}")
        print(f"Final Right Encoder Position: {right_encoder_position}")
        print("=" * 50)

if __name__ == '__main__':
    main()
