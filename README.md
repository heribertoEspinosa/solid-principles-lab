# solid-principles-lab

## Problema identificado
El código original de la clase `SystemManager` presenta múltiples responsabilidades, incluyendo el procesamiento de órdenes y la interacción con servicios externos, lo que viola el principio de responsabilidad única (SRP). Además, la clase depende directamente de implementaciones concretas de servicios de pago, violando así el principio de inversión de dependencias (DIP). Por último, el código no está abierto a la extensión, ya que no proporciona una forma fácil de agregar nuevos métodos de procesamiento de pago ni tipos de notificaciones sin modificar la clase `SystemManager`, lo que viola el principio de abierto/cerrado (OCP)


# Código refactorizdo

enum NotificationType {
    EMAIL, SMS
}

class Order {

    private String id;
    private String name;
    private PaymentType type;
    private com.project.ironHack.entites.NotificationType notificationType;
    private double amount;
    private boolean inStock;

}

enum PaymentType {
    STANDARD,EXPRESS
}

class OutOfStockException extends Exception{
    public OutOfStockException(String message){
        super(message);
    }
}

class OrderProcessorFactory {

    public Payment getPayment(com.project.ironHack.entites.Order order){
        if (com.project.ironHack.entites.PaymentType.STANDARD.equals(order.getType())){
            return new StandardPayment();
        } else if(com.project.ironHack.entites.PaymentType.EXPRESS.equals(order.getType())){
            System.out.println("Tipo de pago: "+ order.getType());
            return new ExpressPayment();
        }
        return null;
    }

    public Notification getNotification(com.project.ironHack.entites.Order order){
        if(com.project.ironHack.entites.NotificationType.EMAIL.equals(order.getNotificationType())){
            return new EmailNotification();
        } else if (com.project.ironHack.entites.NotificationType.SMS.equals(order.getNotificationType())) {
            return new SMSNotification();
        }
        return null;
    }
}

class EmailNotification implements Notification {
    @Override
    public void sendNotification(String title, String message) {
        System.out.println("Send email");
    }
}

class ExpressPayment implements Payment {

    @Override
    public boolean process(double amount) {
        return false;
    }
}


class InventoryService implements Inventory {

    @Override
    public void verifyInventory(com.project.ironHack.entites.Order order) throws com.project.ironHack.exceptions.OutOfStockException {
        if(!order.isInStock()){
            throw new com.project.ironHack.exceptions.OutOfStockException("Out of stock");
        }
    }
}

class SMSNotification implements Notification {
    @Override
    public void sendNotification(String title, String message) {
        System.out.println("Send sms");
    }
}

class StandardPayment implements Payment {
    @Override
    public boolean process(double amount) {
        return true;
    }
}

interface Inventory {
    void verifyInventory(com.project.ironHack.entites.Order order) throws com.project.ironHack.exceptions.OutOfStockException;
}

interface Notification {
    void sendNotification(String title, String message);
}

interface Payment {
    boolean process(double amount);
}

class SystemManager {

    public void processOrder(Order order){
        Payment p = orderProcessorFactory.getPayment(order);
        Notification n = orderProcessorFactory.getNotification(order);

        try {
            inventoryService.verifyInventory(order);
        } catch (OutOfStockException e) {
            System.out.println("Error: " + e.getMessage());
        }

        p.process(order.getAmount());
        n.sendNotification("title", "message");
    }

}

## Cambios Realizados

Se realizaron los siguientes cambios en el código para adherirse a los principios SOLID:

### Single Responsibility Principle (SRP)

- **Issue**: La clase `SystemManager` tenía múltiples responsabilidades, incluyendo el procesamiento de órdenes y la interacción con servicios externos.
- **Cómo se abordó**: Se crearon nuevas clases para manejar la lógica de procesamiento de órdenes, pagos, notificaciones y verificación de inventario, separando así las responsabilidades.
- **Beneficios**: Mejoró la cohesión y la facilidad de mantenimiento del código al tener clases que se centran en una única responsabilidad.

### Open/Closed Principle (OCP)

- **Issue**: El código original no estaba abierto a la extensión y cerrado a la modificación, ya que no proporcionaba una forma fácil de agregar nuevos tipos de pago o notificación.
- **Cómo se abordó**: Se introdujeron interfaces y una fábrica para crear instancias de pagos y notificaciones, permitiendo agregar nuevos tipos sin modificar el código existente.
- **Beneficios**: El código es más flexible y extensible, facilitando la incorporación de nuevas funcionalidades y adaptaciones futuras del sistema.

### Dependency Inversion Principle (DIP)

- **Issue**: La clase original tenía dependencias fuertemente acopladas con implementaciones concretas, lo que dificultaba la sustitución y la prueba de componentes individuales.
- **Cómo se abordó**: Se introdujeron interfaces para desacoplar las dependencias de las implementaciones concretas, facilitando la sustitución y la prueba de componentes.
- **Beneficios**: Hace que el código sea más flexible y fácil de mantener, permitiendo cambiar las implementaciones subyacentes sin afectar al código cliente.

![Captura de pantalla 2024-05-15 a la(s) 11 06 53 p m](https://github.com/heribertoEspinosa/solid-principles-lab/assets/126496110/6f582bca-7ca4-4a7a-96b4-d4a8901574ed)



