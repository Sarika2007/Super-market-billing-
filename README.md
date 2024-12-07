# Super-market-billing
include <stdio.h>
#include <stdlib.h>
#include <string.h>

#define MAX_PRODUCTS 100
#define FILENAME "inventory.txt"

// Product structure
typedef struct {
    int id;
    char name[50];
    float price;
    int quantity;
} Product;

// Global variables
Product inventory[MAX_PRODUCTS];
int productCount = 0;

// Function prototypes
void loadInventory();
void saveInventory();
void addProduct();
void displayInventory();
void generateBill();
void updateStock(int id, int qtySold);
void clearBuffer();

int main() {
    int choice;

    // Load inventory from file
    loadInventory();

    while (1) {
        printf("\n==== Supermarket Billing System ====\n");
        printf("1. Add Product to Inventory\n");
        printf("2. Display Inventory\n");
        printf("3. Generate Bill\n");
        printf("4. Exit\n");
        printf("Enter your choice: ");
        scanf("%d", &choice);
        clearBuffer();

        switch (choice) {
            case 1:
                addProduct();
                break;
            case 2:
                displayInventory();
                break;
            case 3:
                generateBill();
                break;
            case 4:
                saveInventory();
                printf("Inventory saved. Exiting program. Goodbye!\n");
                exit(0);
            default:
                printf("Invalid choice. Please try again.\n");
        }
    }
    return 0;
}

// Function to clear input buffer
void clearBuffer() {
    while (getchar() != '\n');
}

// Function to load inventory from file
void loadInventory() {
    FILE *file = fopen(FILENAME, "r");
    if (file == NULL) {
        printf("No existing inventory found. Starting fresh.\n");
        return;
    }

    while (fscanf(file, "%d,%49[^,],%f,%d\n", &inventory[productCount].id, inventory[productCount].name, 
                  &inventory[productCount].price, &inventory[productCount].quantity) == 4) {
        productCount++;
    }
    fclose(file);
    printf("Inventory loaded successfully.\n");
}

// Function to save inventory to file
void saveInventory() {
    FILE *file = fopen(FILENAME, "w");
    if (file == NULL) {
        printf("Error saving inventory.\n");
        return;
    }

    for (int i = 0; i < productCount; i++) {
        fprintf(file, "%d,%s,%.2f,%d\n", inventory[i].id, inventory[i].name, inventory[i].price, inventory[i].quantity);
    }
    fclose(file);
}

// Function to add a new product
void addProduct() {
    if (productCount >= MAX_PRODUCTS) {
        printf("Inventory is full. Cannot add more products.\n");
        return;
    }

    Product newProduct;
    printf("Enter Product ID: ");
    scanf("%d", &newProduct.id);
    clearBuffer();
    printf("Enter Product Name: ");
    fgets(newProduct.name, sizeof(newProduct.name), stdin);
    strtok(newProduct.name, "\n");
    printf("Enter Product Price: ");
    scanf("%f", &newProduct.price);
    printf("Enter Product Quantity: ");
    scanf("%d", &newProduct.quantity);

    inventory[productCount++] = newProduct;
    printf("Product added successfully!\n");
}

// Function to display inventory
void displayInventory() {
    if (productCount == 0) {
        printf("Inventory is empty.\n");
        return;
    }

    printf("\n==== Inventory List ====\n");
    printf("%-10s %-30s %-10s %-10s\n", "ID", "Name", "Price", "Quantity");
    for (int i = 0; i < productCount; i++) {
        printf("%-10d %-30s %-10.2f %-10d\n", inventory[i].id, inventory[i].name, inventory[i].price, inventory[i].quantity);
    }
}

// Function to generate a bill
void generateBill() {
    if (productCount == 0) {
        printf("Inventory is empty. No products available for billing.\n");
        return;
    }

    int productId, quantity;
    char more = 'y';
    float total = 0;

    printf("\n==== Generate Bill ====\n");
    printf("%-10s %-30s %-10s %-10s\n", "ID", "Name", "Price", "Total");

    while (more == 'y' || more == 'Y') {
        printf("Enter Product ID: ");
        scanf("%d", &productId);
        printf("Enter Quantity: ");
        scanf("%d", &quantity);

        int found = 0;
        for (int i = 0; i < productCount; i++) {
            if (inventory[i].id == productId) {
                found = 1;
                if (inventory[i].quantity >= quantity) {
                    float cost = inventory[i].price * quantity;
                    printf("%-10d %-30s %-10.2f %-10.2f\n", inventory[i].id, inventory[i].name, inventory[i].price, cost);
                    total += cost;
                    updateStock(productId, quantity);
                } else {
                    printf("Insufficient stock for product %s.\n", inventory[i].name);
                }
                break;
            }
        }

        if (!found) {
            printf("Product ID %d not found in inventory.\n", productId);
        }

        printf("Do you want to add more items to the bill? (y/n): ");
        clearBuffer();
        scanf("%c", &more);
    }

    printf("\n==== Total Bill Amount: %.2f ====\n", total);
}

// Function to update inventory stock
void updateStock(int id, int qtySold) {
    for (int i = 0; i < productCount; i++) {
        if (inventory[i].id == id) {
            inventory[i].quantity -= qtySold;
            break;
        }
    }
}
