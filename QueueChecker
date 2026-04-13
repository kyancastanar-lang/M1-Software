import java.sql.*;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.util.concurrent.Executors;
import java.util.concurrent.ScheduledExecutorService;
import java.util.concurrent.TimeUnit;

public class QueueChecker {

    static Repository repo = Repository.getInstance();

    private final ScheduledExecutorService scheduler;

    public QueueChecker() {
        this.repo = Repository.getInstance();
        this.scheduler = Executors.newSingleThreadScheduledExecutor();
    }

    public void checkInClass(int customerID, String className) {
        String membership = repo.getMembershipType(customerID);
        int position = repo.getQueuePosition(membership);

        System.out.println("\n=== CHECK-IN CONFIRMED ===");
        System.out.println("Class: " + className);

        if ("VIP".equalsIgnoreCase(membership)) {
            System.out.println("★ VIP Access | Immediate Entry");
            repo.saveQueueEntry(customerID, className, "VIP", 0, "ACTIVE");
        } else {
            System.out.println(">> Regular Queue | Position: " + position);
            System.out.println("Estimated wait: " + position + " minutes");
            repo.saveQueueEntry(customerID, className, "REGULAR", position, "WAITING");
            startQueueTimer(customerID, position);
        }
    }

    private boolean isAlreadyInQueue(int customerID) {
        String sql = "SELECT COUNT(*) FROM tbl_queue WHERE customerID = ? AND status IN ('WAITING', 'ACTIVE')";
        try (Connection conn = DriverManager.getConnection(repo.getDbURL());
             PreparedStatement pstmt = conn.prepareStatement(sql)) {
            pstmt.setInt(1, customerID);
            ResultSet rs = pstmt.executeQuery();
            if (rs.next()) return rs.getInt(1) > 0;
        } catch (SQLException e) {
            System.out.println("Database error: " + e.getMessage());
        }
        return false;
    }

    private void startQueueTimer(int customerID, int position) {
        scheduler.schedule(() -> {
            repo.updateQueueStatus(customerID, "READY");
            System.out.println("\n>>> Customer " + customerID + " - Your class is ready!");
        }, position, TimeUnit.MINUTES);
    }

    public void showQueueStatus(int customerID) {
        String membership = repo.getMembershipType(customerID);
        System.out.println("\n=== QUEUE STATUS ===");

        if ("VIP".equalsIgnoreCase(membership)) {
            System.out.println("★ VIP Member - Priority Access Enabled");
            return;
        }

        String[] queueData = repo.getQueueStatus(customerID);
        if (queueData != null) {
            String className = queueData[0];
            String status    = queueData[1];
            int position     = Integer.parseInt(queueData[2]);

            if ("WAITING".equals(status) || "ACTIVE".equals(status)) {
                System.out.println("Class   : " + className);
                System.out.println("Position: " + position);
                System.out.println("Status  : " + status);
                if ("WAITING".equals(status))
                    System.out.println("Estimated wait: " + position + " minutes");
            } else {
                System.out.println("No active queue entries.");
            }
        } else {
            System.out.println("No active queue entries.");
        }
    }

    public void shutdown() {
        scheduler.shutdown();
    }
}
