import java.util.ArrayList;
import java.util.List;
import java.util.Scanner;

public class Main {
    private static final Scanner sc = new Scanner(System.in);

    public static void main(String[] args) {
        InventoryManager manager = new InventoryManager();

        // Sample data
        manager.addProduct("Laptop", 45000.00, 12, 5);
        manager.addProduct("Wireless Mouse", 450.00, 4, 10);
        manager.addProduct("Keyboard", 900.00, 25, 8);
        manager.addSupplier("TechSource Pvt Ltd", "9876543210");
        manager.addSupplier("Global Peripherals", "9123456780");

        boolean running = true;
        while (running) {
            System.out.println("\n===== INVENTORY & PURCHASE ORDERS =====");
            System.out.println("1. View inventory");
            System.out.println("2. View low-stock items");
            System.out.println("3. Add product");
            System.out.println("4. Add supplier");
            System.out.println("5. Create purchase order");
            System.out.println("6. View purchase orders");
            System.out.println("7. Receive purchase order");
            System.out.println("8. Cancel purchase order");
            System.out.println("9. Exit");

            int choice = readInt("Choose an option: ");

            switch (choice) {
                case 1:
                    printList(manager.getProducts(), "No products in inventory.");
                    break;
                case 2:
                    printList(manager.getLowStockProducts(), "No low-stock items.");
                    break;
                case 3:
                    addProduct(manager);
                    break;
                case 4:
                    addSupplier(manager);
                    break;
                case 5:
                    createOrder(manager);
                    break;
                case 6:
                    printList(manager.getOrders(), "No purchase orders yet.");
                    break;
                case 7:
                    System.out.print("PO number to receive: ");
                    System.out.println(manager.receiveOrder(sc.nextLine().trim())
                            ? "Order received. Stock updated."
                            : "Order not found or not pending.");
                    break;
                case 8:
                    System.out.print("PO number to cancel: ");
                    System.out.println(manager.cancelOrder(sc.nextLine().trim())
                            ? "Order cancelled."
                            : "Order not found or not pending.");
                    break;
                case 9:
                    running = false;
                    System.out.println("Goodbye!");
                    break;
                default:
                    System.out.println("Invalid option.");
            }
        }
        sc.close();
    }

    private static void addProduct(InventoryManager manager) {
        System.out.print("Product name: ");
        String name = sc.nextLine().trim();
        double cost = readDouble("Unit cost: ");
        int qty = readInt("Opening stock: ");
        int reorder = readInt("Reorder level: ");

        if (name.isEmpty() || cost < 0 || qty < 0 || reorder < 0) {
            System.out.println("Invalid product details.");
            return;
        }
        System.out.println("Added: " + manager.addProduct(name, cost, qty, reorder));
    }

    private static void addSupplier(InventoryManager manager) {
        System.out.print("Supplier name: ");
        String name = sc.nextLine().trim();
        System.out.print("Phone: ");
        String phone = sc.nextLine().trim();

        if (name.isEmpty()) {
            System.out.println("Name cannot be empty.");
            return;
        }
        System.out.println("Added: " + manager.addSupplier(name, phone));
    }

    private static void createOrder(InventoryManager manager) {
        if (manager.getSuppliers().isEmpty() || manager.getProducts().isEmpty()) {
            System.out.println("You need at least one supplier and one product first.");
            return;
        }

        manager.getSuppliers().forEach(System.out::println);
        int supplierId = readInt("Supplier ID: ");
        if (manager.findSupplier(supplierId) == null) {
            System.out.println("Supplier not found.");
            return;
        }

        List<OrderItem> items = new ArrayList<>();
        manager.getProducts().forEach(System.out::println);
        System.out.println("Add items to the order (enter product ID 0 to finish).");

        while (true) {
            int productId = readInt("Product ID: ");
            if (productId == 0) break;

            Product p = manager.findProduct(productId);
            if (p == null) {
                System.out.println("Product not found.");
                continue;
            }
            int qty = readInt("Quantity: ");
            if (qty <= 0) {
                System.out.println("Quantity must be positive.");
                continue;
            }
            items.add(new OrderItem(p, qty));
            System.out.println("Added " + qty + " x " + p.getName());
        }

        PurchaseOrder po = manager.createOrder(supplierId, items);
        if (po == null) {
            System.out.println("Order not created: no items added.");
        } else {
            System.out.println("Purchase order created:");
            System.out.print(po);
        }
    }

    private static <T> void printList(List<T> list, String emptyMessage) {
        if (list.isEmpty()) {
            System.out.println(emptyMessage);
        } else {
            list.forEach(System.out::println);
        }
    }

    // Keeps asking until the user enters a valid whole number
    private static int readInt(String prompt) {
        while (true) {
            System.out.print(prompt);
            try {
                return Integer.parseInt(sc.nextLine().trim());
            } catch (NumberFormatException e) {
                System.out.println("Please enter a valid number.");
            }
        }
    }

    // Keeps asking until the user enters a valid decimal number
    private static double readDouble(String prompt) {
        while (true) {
            System.out.print(prompt);
            try {
                return Double.parseDouble(sc.nextLine().trim());
            } catch (NumberFormatException e) {
                System.out.println("Please enter a valid number.");
            }
        }
    }
}
import java.util.ArrayList;
import java.util.List;

public class InventoryManager {
    private final List<Product> products = new ArrayList<>();
    private final List<Supplier> suppliers = new ArrayList<>();
    private final List<PurchaseOrder> orders = new ArrayList<>();
    private int nextProductId = 1;
    private int nextSupplierId = 1;
    private int nextPoNumber = 5001;

    // ---------- Products ----------
    public Product addProduct(String name, double unitCost, int quantity, int reorderLevel) {
        Product p = new Product(nextProductId++, name, unitCost, quantity, reorderLevel);
        products.add(p);
        return p;
    }

    public List<Product> getProducts() { return products; }

    public Product findProduct(int id) {
        for (Product p : products) {
            if (p.getId() == id) return p;
        }
        return null;
    }

    public List<Product> getLowStockProducts() {
        List<Product> low = new ArrayList<>();
        for (Product p : products) {
            if (p.isLowStock()) low.add(p);
        }
        return low;
    }

    // ---------- Suppliers ----------
    public Supplier addSupplier(String name, String phone) {
        Supplier s = new Supplier(nextSupplierId++, name, phone);
        suppliers.add(s);
        return s;
    }

    public List<Supplier> getSuppliers() { return suppliers; }

    public Supplier findSupplier(int id) {
        for (Supplier s : suppliers) {
            if (s.getId() == id) return s;
        }
        return null;
    }

    // ---------- Purchase Orders ----------
    public PurchaseOrder createOrder(int supplierId, List<OrderItem> items) {
        Supplier supplier = findSupplier(supplierId);
        if (supplier == null || items.isEmpty()) return null;

        PurchaseOrder po = new PurchaseOrder("PO" + nextPoNumber++, supplier, items);
        orders.add(po);
        return po;
    }

    public List<PurchaseOrder> getOrders() { return orders; }

    public PurchaseOrder findOrder(String poNumber) {
        for (PurchaseOrder po : orders) {
            if (po.getPoNumber().equalsIgnoreCase(poNumber)) return po;
        }
        return null;
    }

    // Receiving a pending order adds the ordered quantities to stock
    public boolean receiveOrder(String poNumber) {
        PurchaseOrder po = findOrder(poNumber);
        if (po == null || po.getStatus() != PurchaseOrder.Status.PENDING) return false;

        for (OrderItem item : po.getItems()) {
            item.getProduct().addStock(item.getQuantity());
        }
        po.markReceived();
        return true;
    }

    public boolean cancelOrder(String poNumber) {
        PurchaseOrder po = findOrder(poNumber);
        if (po == null || po.getStatus() != PurchaseOrder.Status.PENDING) return false;
        po.cancel();
        return true;
    }
}
import java.util.ArrayList;
import java.util.List;

public class PurchaseOrder {
    public enum Status { PENDING, RECEIVED, CANCELLED }

    private final String poNumber;
    private final Supplier supplier;
    private final List<OrderItem> items;
    private Status status;

    public PurchaseOrder(String poNumber, Supplier supplier, List<OrderItem> items) {
        this.poNumber = poNumber;
        this.supplier = supplier;
        this.items = new ArrayList<>(items);
        this.status = Status.PENDING;
    }

    public String getPoNumber() { return poNumber; }
    public Supplier getSupplier() { return supplier; }
    public List<OrderItem> getItems() { return items; }
    public Status getStatus() { return status; }

    public double getTotalAmount() {
        double total = 0;
        for (OrderItem item : items) total += item.getLineTotal();
        return total;
    }

    public void markReceived() { status = Status.RECEIVED; }
    public void cancel() { status = Status.CANCELLED; }

    @Override
    public String toString() {
        StringBuilder sb = new StringBuilder();
        sb.append(String.format("%s | Supplier: %s | Status: %s | Total: Rs.%.2f%n",
                poNumber, supplier.getName(), status, getTotalAmount()));
        for (OrderItem item : items) sb.append(item).append("\n");
        return sb.toString();
    }
}
public class OrderItem {
    private final Product product;
    private final int quantity;

    public OrderItem(Product product, int quantity) {
        this.product = product;
        this.quantity = quantity;
    }

    public Product getProduct() { return product; }
    public int getQuantity() { return quantity; }

    public double getLineTotal() {
        return product.getUnitCost() * quantity;
    }

    @Override
    public String toString() {
        return String.format("   - %-18s x %-5d @ Rs.%-8.2f = Rs.%.2f",
                product.getName(), quantity, product.getUnitCost(), getLineTotal());
    }
}
public class Product {
    private final int id;
    private final String name;
    private final double unitCost;
    private final int reorderLevel;
    private int quantity;

    public Product(int id, String name, double unitCost, int quantity, int reorderLevel) {
        this.id = id;
        this.name = name;
        this.unitCost = unitCost;
        this.quantity = quantity;
        this.reorderLevel = reorderLevel;
    }

    public int getId() { return id; }
    public String getName() { return name; }
    public double getUnitCost() { return unitCost; }
    public int getQuantity() { return quantity; }
    public int getReorderLevel() { return reorderLevel; }

    public void addStock(int amount) {
        if (amount > 0) quantity += amount;
    }

    public boolean isLowStock() {
        return quantity <= reorderLevel;
    }

    @Override
    public String toString() {
        return String.format("[%d] %-18s | Cost: Rs.%-8.2f | Stock: %-5d | Reorder at: %d%s",
                id, name, unitCost, quantity, reorderLevel, isLowStock() ? "  <-- LOW STOCK" : "");
    }
}
public class Supplier {
    private final int id;
    private final String name;
    private final String phone;

    public Supplier(int id, String name, String phone) {
        this.id = id;
        this.name = name;
        this.phone = phone;
    }

    public int getId() { return id; }
    public String getName() { return name; }
    public String getPhone() { return phone; }

    @Override
    public String toString() {
        return String.format("[%d] %-20s | Phone: %s", id, name, phone);
    }
}
