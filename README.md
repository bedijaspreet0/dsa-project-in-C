# dsa-project-in-C
Topic-Online shopping Cart
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

// Structure to define a Product available in the inventory
typedef struct Product {
    int id;
    char name[50];
    float price;
    int stock;
    struct Product* next;
} Product;

// Structure to define an Item added to the Shopping Cart
typedef struct CartItem {
    int productId;
    char name[50];
    float price;
    int quantity;
    struct CartItem* next;
} CartItem;

// Global head pointers for Data Structures
Product* inventoryHead = NULL;
CartItem* cartHead = NULL;

// Function Prototypes
void initializeInventory();
void displayInventory();
void addToCart(int id, int qty);
void removeFromCart(int id);
void viewCart();
void checkout();
Product* findProduct(int id);

int main() {
    initializeInventory();
    int choice, id, qty;

    while (1) {
        printf("\n=== ONLINE SHOPPING CART SYSTEM (DSA) ===\n");
        printf("1. View Available Products\n");
        printf("2. Add Product to Cart\n");
        printf("3. Remove Product from Cart\n");
        printf("4. View My Cart\n");
        printf("5. Checkout and Generate Receipt\n");
        printf("6. Exit\n");
        printf("Enter your choice: ");
        scanf("%d", &choice);

        switch (choice) {
            case 1:
                displayInventory();
                break;
            case 2:
                printf("Enter Product ID to add: ");
                scanf("%d", &id);
                printf("Enter Quantity: ");
                scanf("%d", &qty);
                addToCart(id, qty);
                break;
            case 3:
                printf("Enter Product ID to remove: ");
                scanf("%d", &id);
                removeFromCart(id);
                break;
            case 4:
                viewCart();
                break;
            case 5:
                checkout();
                break;
            case 6:
                printf("Thank you for shopping with us!\n");
                exit(0);
            default:
                printf("Invalid choice! Please try again.\n");
        }
    }
    return 0;
}

// Pre-populates the inventory linked list with items
void initializeInventory() {
    int ids[] = {101, 102, 103, 104};
    char names[][50] = {"Laptop", "Smartphone", "Headphones", "Smartwatch"};
    float prices[] = {799.99, 499.99, 89.99, 149.99};
    int stocks[] = {10, 20, 30, 15};

    for (int i = 0; i < 4; i++) {
        Product* newProd = (Product*)malloc(sizeof(Product));
        newProd->id = ids[i];
        strcpy(newProd->name, names[i]);
        newProd->price = prices[i];
        newProd->stock = stocks[i];
        newProd->next = inventoryHead;
        inventoryHead = newProd;
    }
}

// Displays all available products in the shop
void displayInventory() {
    Product* temp = inventoryHead;
    printf("\n--- Available Products ---\n");
    printf("ID\tName\t\tPrice\t\tStock\n");
    while (temp != NULL) {
        printf("%d\t%-15s$%.2f\t\t%d\n", temp->id, temp->name, temp->price, temp->stock);
        temp = temp->next;
    }
}

// Helper function to search for a product in the inventory
Product* findProduct(int id) {
    Product* temp = inventoryHead;
    while (temp != NULL) {
        if (temp->id == id) return temp;
        temp = temp->next;
    }
    return NULL;
}

// Adds an item to the shopping cart or updates quantity if it already exists
void addToCart(int id, int qty) {
    Product* prod = findProduct(id);
    if (prod == NULL) {
        printf("Product not found!\n");
        return;
    }
    if (prod->stock < qty) {
        printf("Insufficient stock! Only %d items available.\n", prod->stock);
        return;
    }

    // Check if item already exists in cart
    CartItem* temp = cartHead;
    while (temp != NULL) {
        if (temp->productId == id) {
            if (prod->stock < (temp->quantity + qty)) {
                printf("Cannot add more. Exceeds available stock.\n");
                return;
            }
            temp->quantity += qty;
            printf("Cart updated successfully!\n");
            return;
        }
        temp = temp->next;
    }

    // Insert new item at the beginning of the cart linked list
    CartItem* newItem = (CartItem*)malloc(sizeof(CartItem));
    newItem->productId = id;
    strcpy(newItem->name, prod->name);
    newItem->price = prod->price;
    newItem->quantity = qty;
    newItem->next = cartHead;
    cartHead = newItem;
    printf("Product added to cart!\n");
}

// Removes an item entirely from the shopping cart
void removeFromCart(int id) {
    CartItem* temp = cartHead;
    CartItem* prev = NULL;

    while (temp != NULL && temp->productId != id) {
        prev = temp;
        temp = temp->next;
    }

    if (temp == NULL) {
        printf("Item not found in your cart!\n");
        return;
    }

    if (prev == NULL) {
        cartHead = temp->next;
    } else {
        prev->next = temp->next;
    }
    free(temp);
    printf("Item removed from cart successfully!\n");
}

// Displays the current items in the user's shopping cart
void viewCart() {
    CartItem* temp = cartHead;
    if (temp == NULL) {
        printf("\nYour cart is empty!\n");
        return;
    }
    float totalCartCost = 0;
    printf("\n--- Your Shopping Cart ---\n");
    printf("ID\tName\t\tPrice\t\tQty\tTotal\n");
    while (temp != NULL) {
        float itemTotal = temp->price * temp->quantity;
        totalCartCost += itemTotal;
        printf("%d\t%-15s$%.2f\t\t%d\t$%.2f\n", temp->productId, temp->name, temp->price, temp->quantity, itemTotal);
        temp = temp->next;
    }
    printf("---------------------------------------------------\n");
    printf("Total Amount Payable: $%.2f\n", totalCartCost);
}

// Finalizes transaction, updates real inventory stocks, and clears the cart
void checkout() {
    CartItem* tempCart = cartHead;
    if (tempCart == NULL) {
        printf("\nYour cart is empty! Nothing to checkout.\n");
        return;
    }

    float finalBill = 0;
    printf("\n====== INVOICE RECEIPT ======\n");
    printf("Item\t\tQty\tPrice\tTotal\n");
    
    while (tempCart != NULL) {
        Product* prod = findProduct(tempCart->productId);
        if (prod != NULL) {
            prod->stock -= tempCart->quantity; // Deduct inventory stock
        }
        float cost = tempCart->price * tempCart->quantity;
        finalBill += cost;
        printf("%-15s%d\t$%.2f\t$%.2f\n", tempCart->name, tempCart->quantity, tempCart->price, cost);
        
        CartItem* nextNode = tempCart->next;
        free(tempCart); // Memory cleanup
        tempCart = nextNode;
    }
    cartHead = NULL; // Clear the cart reference
    printf("---------------------------------\n");
    printf("Grand Total Paid: $%.2f\n", finalBill);
    printf("=================================\n");
    printf("Order placed successfully! Checkout complete.\n");
}


