# simulatetennis.com
/*
   Adrian Marin
   3/18/2026
   
   This program predict the porjectile that a tennis ball takes with 
   varrying inital speed, spin, and wind.
*/

import java.util.*; // for Scanner

public class Projectile {
   // constant for earth's gravity
   public static final double acceleration = -9.81;
   
   //coefficients for air resistance
   public static final double airDensity = 1.225; //kg/m^3
   public static final double drag = 0.507; //drag of a babolat all court ball
   public static final double ballMass = 0.056; // in kg
   public static final double ballRadius = 0.0327; //in meters
   
   //Magnus effect constants
   public static final double liftCoefficient = 0.25;
   public static final double magnusConstant = 0.5 * airDensity * liftCoefficient * (Math.PI * ballRadius * ballRadius) / ballMass;
   
   //derviing the drag constant
   public static final double dragConstant = 0.5 * airDensity * drag * (Math.PI * ballRadius * ballRadius) / ballMass;
   
   //set up main method
   public static void main(String[] args) {
      Scanner scan = new Scanner(System.in);
      
      //Tell user the point of program
      giveIntro();
            
      //prompt user for information
      System.out.print("What is the initial velocity in meters per sec? ");
      double initVel = scan.nextDouble();
      System.out.print("What is the angle in degree above the horizontal? ");
      double angle = Math.toRadians(scan.nextDouble());
      System.out.print("What is the number of steps that you would like to be displayed? ");
      int steps = scan.nextInt();
      System.out.print("What was the initial height in meters? ");
      double initHeight = scan.nextDouble();
      System.out.print("What is the topspin in RPM? (negative for backspin): ");
      double spinRPM = scan.nextDouble();
      double omega = spinRPM * (2 * Math.PI / 60); // convert RPM to rad/s
      System.out.println();
      
      //call printTable()
      printTable(initVel, angle, steps, initHeight, omega);
      
   }
   
   //round2 method round the number
   public static double round2(double n) {
      return Math.round(n * 100.0) / 100.0;
   }
   
   //give intro mehtod introduces the program
   public static void giveIntro() {
      System.out.println("This program computes the ");
      System.out.print("trajectory of a tennis ball given: ");
      System.out.println("its inital velocity, its angle relative to the horizontal, ");
      System.out.print("the amount of spin, and the wind.");
      System.out.println();
   }
   
   //calculates the displacement using the formula vt+0.5at^2
   public static double estimateFlightTime(double vx, double vy, double initHeight, double omega) {
      double x = 0.0;
      double y = initHeight;
      double dt = 0.0001;
      double t = 0.0;
      
      //while loop for drag at all dt's
      while (y >= 0) {
         double speed = Math.sqrt(vx * vx + vy * vy);
         
         //drag in x and y
         double dragX = -dragConstant * speed * vx; //x component of the drag
         double dragY = -dragConstant * speed * vy;
         
         //Magnus forse: topspin has postive omega and backspin negative
         double magnusY = -magnusConstant * omega * vx; // cross product: ω × v in 2D
         double magnusX =  magnusConstant * omega * vy;
         
         //update the velocities using euler integration
         vx += (dragX + magnusX) * dt;
         vy += (acceleration + dragY + magnusY) * dt; // gravity acts downward on vy
         
         //update postion of object
         x += vx * dt;
         y += vy * dt;
         t += dt;      
      }
      return t;
   }
   
   //calculates the x and y positions as time passes and print the table
   public static void printTable(double initVel, double angle, int steps, double initHeight, double omega) {
      double vx = initVel * Math.cos(angle);
      double vy = initVel * Math.sin(angle);
      double totalTime = estimateFlightTime(vx, vy, initHeight, omega);
      double dt = 0.0001;
      double printInterval = totalTime / steps;
      double nextPrintTime = printInterval; // FIX 1: was 0.0, caused step 1 to fire immediately
             
      double x = 0.0;
      double y = initHeight;
      double t = 0.0;
      int step = 0;
      
      System.out.println("s\tx\ty\ttime");
      System.out.println("0\t0.0\t" + initHeight + "\t0.0");
       
       //Find x and y corrdinates at each step
      while (y >= 0 && step < steps) {
         double speed = Math.sqrt(vx * vx + vy * vy);
      
         double dragX = -dragConstant * speed * vx;
         double dragY = -dragConstant * speed * vy;
      
         //Magnus forse: topspin has postive omega and backspin negative
         double magnusY = -magnusConstant * omega * vx;
         double magnusX =  magnusConstant * omega * vy;
      
         vx += (dragX + magnusX) * dt;
         vy += (acceleration + dragY + magnusY) * dt;
         
         //update postion of object // FIX 2: moved before print check so x and y are updated before printing
         x += vx * dt;
         y += vy * dt;
         t += dt;
         
         // Print at evenly spaced intervals
         if (t >= nextPrintTime - dt / 2) {
            step++;
            System.out.println(step + "\t" + round2(x) + "\t" + round2(y) + "\t" + round2(t)); // FIX 3: removed forced y=0 on last step
            nextPrintTime += printInterval;
         }
      }
   }
}
