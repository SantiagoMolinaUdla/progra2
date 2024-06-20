abstract class Entity {
    protected int id;
    protected String type;

    public Entity(int id, String type) {
        this.id = id;
        this.type = type;
    }

    public abstract void display();
}

interface Parkable {
    void park(Car car);
    void remove();
    boolean isAvailable();
}

class Car extends Entity {
    private String licensePlate;
    private int entryHour;

    public Car(int id, String licensePlate, int entryHour) {
        super(id, "Car");
        this.licensePlate = licensePlate;
        this.entryHour = entryHour;
    }

    public String getLicensePlate() {
        return licensePlate;
    }

    public int getEntryHour() {
        return entryHour;
    }

    public void display() {
        System.out.println("Car ID: " + id + ", License Plate: " + licensePlate + ", Entry Hour: " + entryHour);
    }
}

class ParkingSpot extends Entity implements Parkable {
    private Car parkedCar;

    public ParkingSpot(int id) {
        super(id, "ParkingSpot");
        this.parkedCar = null;
    }

    public void park(Car car) {
        this.parkedCar = car;
    }

    public void remove() {
        this.parkedCar = null;
    }

    public boolean isAvailable() {
        return parkedCar == null;
    }

    public Car getParkedCar() {
        return parkedCar;
    }

    public void display() {
        System.out.println("Parking Spot ID: " + id + ", Available: " + isAvailable());
    }
}

public class ParkingLot {
    private static final int OPENING_HOUR = 6;
    private static final int CLOSING_HOUR = 21;

    private ParkingSpot[] spots;
    private int currentHour;
    private double tariff;
    private double earnings;

    public ParkingLot() {
        spots = new ParkingSpot[5];
        for (int i = 0; i < 5; i++) {
            spots[i] = new ParkingSpot(i + 1);
        }
        currentHour = OPENING_HOUR;
        tariff = 0;
        earnings = 0;
    }

    public void setInitialTariff(double tariff) {
        this.tariff = tariff;
    }

    public void setInitialHour(int hour) {
        if (hour >= OPENING_HOUR && hour <= CLOSING_HOUR) {
            this.currentHour = hour;
        } else {
            System.out.println("Hora inicial inválida. Debe estar entre " + OPENING_HOUR + " y " + CLOSING_HOUR);
        }
    }

    public void parkCar(String licensePlate, int entryHour) {
        if (entryHour < OPENING_HOUR || entryHour > CLOSING_HOUR) {
            System.out.println("Hora de entrada inválida. Debe estar entre " + OPENING_HOUR + " y " + CLOSING_HOUR);
            return;
        }

        for (int i = 0; i < spots.length; i++) {
            if (!spots[i].isAvailable() && spots[i].getParkedCar().getLicensePlate().equals(licensePlate)) {
                System.out.println("Ya hay un carro con la misma placa en el parqueadero.");
                return;
            }
        }

        for (int i = 0; i < spots.length; i++) {
            if (spots[i].isAvailable()) {
                spots[i].park(new Car(spots[i].id, licensePlate, entryHour));
                System.out.println("Carro con placa " + licensePlate + " parqueado en el puesto " + spots[i].id);
                return;
            }
        }

        System.out.println("No hay puestos disponibles.");
    }

    public void removeCar(String licensePlate) {
        for (int i = 0; i < spots.length; i++) {
            if (!spots[i].isAvailable() && spots[i].getParkedCar().getLicensePlate().equals(licensePlate)) {
                int hoursParked = currentHour - spots[i].getParkedCar().getEntryHour();
                if (hoursParked == 0) {
                    hoursParked = 1;  // Se cobra al menos una hora
                }
                double cost = hoursParked * tariff;
                earnings += cost;
                spots[i].remove();
                System.out.println("Carro con placa " + licensePlate + " salió del parqueadero. Costo: $" + cost);
                return;
            }
        }

        System.out.println("No se encontró un carro con la placa " + licensePlate);
    }

    public double getEarnings() {
        return earnings;
    }

    public int getAvailableSpots() {
        int count = 0;
        for (int i = 0; i < spots.length; i++) {
            if (spots[i].isAvailable()) {
                count++;
            }
        }
        return count;
    }

    public void advanceTime(int hours) {
        if (currentHour + hours <= CLOSING_HOUR) {
            currentHour += hours;
            System.out.println("Hora actual: " + currentHour);
        } else {
            System.out.println("No se puede avanzar el tiempo más allá de la hora de cierre.");
        }
    }

    public void changeTariff(double newTariff) {
        tariff = newTariff;
    }

    public static void main(String[] args) {
        ParkingLot parkingLot = new ParkingLot();
        java.util.Scanner scanner = new java.util.Scanner(System.in);

        while (true) {
            System.out.println("\n--- Menú de Parqueadero ---");
            System.out.println("1. Ingresar la tarifa inicial");
            System.out.println("2. Ingresar la hora inicial");
            System.out.println("3. Ingresar un carro al parqueadero");
            System.out.println("4. Dar salida a un carro del parqueadero");
            System.out.println("5. Informar los ingresos del parqueadero");
            System.out.println("6. Consultar la cantidad de puestos disponibles");
            System.out.println("7. Avanzar el reloj del parqueadero");
            System.out.println("8. Cambiar la tarifa del parqueadero");
            System.out.println("9. Salir");
            System.out.print("Seleccione una opción: ");

            int option = scanner.nextInt();
            switch (option) {
                case 1:
                    System.out.print("Ingrese la tarifa inicial: ");
                    double tariff = scanner.nextDouble();
                    parkingLot.setInitialTariff(tariff);
                    break;
                case 2:
                    System.out.print("Ingrese la hora inicial: ");
                    int initialHour = scanner.nextInt();
                    parkingLot.setInitialHour(initialHour);
                    break;
                case 3:
                    System.out.print("Ingrese la placa del carro: ");
                    String licensePlate = scanner.next();
                    System.out.print("Ingrese la hora de entrada: ");
                    int entryHour = scanner.nextInt();
                    parkingLot.parkCar(licensePlate, entryHour);
                    break;
                case 4:
                    System.out.print("Ingrese la placa del carro: ");
                    licensePlate = scanner.next();
                    parkingLot.removeCar(licensePlate);
                    break;
                case 5:
                    System.out.println("Ingresos del parqueadero: $" + parkingLot.getEarnings());
                    break;
                case 6:
                    System.out.println("Cantidad de puestos disponibles: " + parkingLot.getAvailableSpots());
                    break;
                case 7:
                    System.out.print("Ingrese la cantidad de horas a avanzar: ");
                    int hours = scanner.nextInt();
                    parkingLot.advanceTime(hours);
                    break;
                case 8:
                    System.out.print("Ingrese la nueva tarifa: ");
                    double newTariff = scanner.nextDouble();
                    parkingLot.changeTariff(newTariff);
                    break;
                case 9:
                    System.out.println("Saliendo...");
                    scanner.close();
                    return;
                default:
                    System.out.println("Opción inválida. Intente nuevamente.");
            }
        }
    }
}