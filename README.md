# Attendance Management System

    import java.sql.*;
    import java.util.*;

    public class Main {
    static Scanner sc = new Scanner(System.in);

    public static void main(String[] args) {
        if (!DatabaseConnection.connect()) {
            System.out.println("Exiting program due to database connection failure.");
            return;
        }

        int choice;
        do {
            System.out.println("\n===== Attendance Management System =====");
            System.out.println("1. Add Student");
            System.out.println("2. Mark Attendance");
            System.out.println("3. View Attendance");
            System.out.println("4. Exit");
            System.out.print("Enter choice: ");
            while (!sc.hasNextInt()) {
                System.out.print("Please enter a valid number: ");
                sc.next();
            }
            choice = sc.nextInt();

            switch (choice) {
                case 1:
                    addStudent();
                    break;
                case 2:
                    markAttendance();
                    break;
                case 3:
                    viewAttendance();
                    break;
                case 4:
                    System.out.println("Exiting... Goodbye!");
                    break;
                default:
                    System.out.println("Invalid choice! Try again.");
            }
        } while (choice != 4);

        DatabaseConnection.close();
    }

    static void addStudent() {
        sc.nextLine(); // consume leftover newline
        System.out.print("Enter student name: ");
        String name = sc.nextLine();

        try {
            PreparedStatement pst = DatabaseConnection.conn.prepareStatement("INSERT INTO students(name) VALUES(?)");
            pst.setString(1, name);
            pst.executeUpdate();
            System.out.println("Student added successfully!");
        } catch (SQLException e) {
            System.out.println("Error adding student: " + e.getMessage());
        }
    }

    static void markAttendance() {
        System.out.print("Enter student ID: ");
        while (!sc.hasNextInt()) {
            System.out.print("Please enter a valid student ID: ");
            sc.next();
        }
        int id = sc.nextInt();
        sc.nextLine(); // consume leftover newline

        System.out.print("Enter date (YYYY-MM-DD): ");
        String date = sc.nextLine();

        try {
            PreparedStatement pst = DatabaseConnection.conn
                    .prepareStatement("INSERT INTO attendance(student_id, date) VALUES(?, ?)");
            pst.setInt(1, id);
            pst.setString(2, date);
            pst.executeUpdate();
            System.out.println("Attendance marked!");
        } catch (SQLException e) {
            System.out.println("Error marking attendance: " + e.getMessage());
        }
    }

    static void viewAttendance() {
        try {
            Statement st = DatabaseConnection.conn.createStatement();
            ResultSet rs = st.executeQuery(
                    "SELECT s.id, s.name, a.date FROM students s JOIN attendance a ON s.id = a.student_id ORDER BY a.date DESC");

            System.out.println("\nID | Name | Date");
            System.out.println("---------------------------");
            while (rs.next()) {
                System.out.printf("%d | %s | %s\n", rs.getInt("id"), rs.getString("name"), rs.getString("date"));
            }
        } catch (SQLException e) {
            System.out.println("Error viewing attendance: " + e.getMessage());
        }
    }
    }

    class DatabaseConnection {
    static Connection conn;

    static boolean connect() {
        try {
            conn = DriverManager.getConnection("jdbc:mysql://localhost:3306/attendancedb", "root", "");
            System.out.println("Connected to database!");
            return true;
        } catch (SQLException e) {
            System.out.println("Database connection error: " + e.getMessage());
            conn = null;
            return false;
        }
    }

    static void close() {
        try {
            if (conn != null && !conn.isClosed()) {
                conn.close();
                System.out.println("Database connection closed.");
            }
        } catch (SQLException e) {
            System.out.println("Error closing connection: " + e.getMessage());
        }
    }
}
