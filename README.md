Mini Hotel Reservation System
A console-based Hotel Management System developed in C++ to automate room bookings, manage customer accounts, and handle admin operations with secure file handling and robust validation.

 Features & Functionalities
Data Management: Utilizes structs (date, customer, room, reservation, review, admin) to manage 5 floors (50 rooms) with dynamic pricing and offers.

Authentication: Distinct portals for Customers (sign up, book, review) and Admins (manage, modify, or cancel bookings).

Core Booking Logic: Prevents double-booking via dynamic date-overlap checks and calculates total costs including extra services.

Data Persistence: Uses File I/O to save/load all records (.txt), ensuring zero data loss between sessions.

 Edge Cases Handled
File Safety: Seamlessly creates missing files upon exit without crashing the system.

Date Validation: Uses <ctime> to block past bookings and ensures chronological date ranges.

Payment Security: Logically validates credit cards using the Luhn Algorithm.

Input Constraints: Prevents duplicate usernames and ensures valid email formats (checks for @).

Review System: Restricts reviews to verified guests and filters out reviews older than 30 days.

Admin Override: Admins can bypass payment validation to manually assist customers.























































#include <iostream>
#include <string>
#include <ctime>
#include <fstream>
#include <cctype>
using namespace std;

const int FLOORS = 5;
const int ROOMS_PER_FLOOR = 10;
const long long ID_OFFSET = 20251700000;
const int MAX_RESERVATIONS = 300;

string toLower(string s)
{
    for (int i = 0; i < s.size(); i++)
        s[i] = tolower(s[i]);
    return s;
}

struct date
{
    int day;
    int month;
    int year;
};

struct customer
{
    string name;
    long long customerId;
    string email;
    string username;
    string password;
    string creditCardNumber;
    int roomNumber;
    date startDate;
    date endDate;
};

struct room
{
    int roomNumber;
    string type;
    double price;
    bool hasOffer;
};

struct reservation
{
    int roomNumber;
    long long customerId;
    date startDate;
    date endDate;
    string creditCardNumber;
    double totalCost;
    bool active;
    bool cancelled;
};

struct review
{
    int roomNumber;
    long long customerId;
    string reviewContent;
    date dateOfReview;
};

struct admin
{
    int adminID;
    string username;
    string password;
};

customer customers[100];
room rooms[FLOORS][ROOMS_PER_FLOOR];
reservation reservations[MAX_RESERVATIONS];
review reviews[10];
admin admins[5];

int CustomerCount = 0;
int reviewCount = 0;
int reservationCount = 0;
int currentUserIndex = -1;

void saveData()
{
    ofstream outCust("customers.txt");
    ofstream outRes("reservations.txt");
    ofstream outRev("reviews.txt");

    // لو ملف مفاتحش لأي سبب بنوقف عشان البرنامج ميضربش
    if (!outCust || !outRes || !outRev)
    {
        cout << "\nerror while opening files\n";
        return;
    }

    // save customers
    outCust << CustomerCount << endl; //  بنسجل العدد الأول عشان نعرف هنقرا كام سطر لما نفتح
    for (int i = 0; i < CustomerCount; i++)
    {
        outCust << customers[i].username << endl;
        outCust << customers[i].password << endl;
        outCust << customers[i].name << endl;
        outCust << customers[i].customerId << " " << customers[i].roomNumber << " "
            << customers[i].startDate.day << " " << customers[i].startDate.month << " " << customers[i].startDate.year << " "
            << customers[i].endDate.day << " " << customers[i].endDate.month << " " << customers[i].endDate.year << " "
            << customers[i].creditCardNumber << endl;
    }

    // save reservations
    outRes << reservationCount << endl;
    for (int i = 0; i < reservationCount; i++)
    {
        outRes << reservations[i].roomNumber << " " << reservations[i].customerId << " " << reservations[i].active << " "
            << reservations[i].cancelled << " " << reservations[i].startDate.day << " " << reservations[i].startDate.month << " " << reservations[i].startDate.year << " "
            << reservations[i].endDate.day << " " << reservations[i].endDate.month << " " << reservations[i].endDate.year << " "
            << reservations[i].creditCardNumber << endl;
    }

    // save reviews

    outRev << reviewCount << endl;
    for (int i = 0; i < reviewCount; i++)
    {
        outRev << reviews[i].customerId << " " << reviews[i].roomNumber << " "
            << reviews[i].dateOfReview.day << " " << reviews[i].dateOfReview.month << " " << reviews[i].dateOfReview.year << endl;
        outRev << reviews[i].reviewContent << endl; // المراجعة في سطر لوحدها عشان المسافات
    }

    // بيقفل الفايل بنفسه علشان اضمن ان الداتا وصلت فعلا لفايل
    outCust.close();
    outRes.close();
    outRev.close();
}

void loadData()
{
    ifstream inCust("customers.txt");
    ifstream inRes("reservations.txt");
    ifstream inRev("reviews.txt");

    //  وبيعمله لوحده  لو الملف مش موجود البرنامج بيكمل عادي ومبيطلعش ايرور
    // Load Customers
    if (inCust)
    {
        inCust >> CustomerCount;
        for (int i = 0; i < CustomerCount; i++)
        {
            inCust >> customers[i].username;
            inCust >> customers[i].password;
            inCust.ignore();
            getline(inCust, customers[i].name);
            inCust >> customers[i].customerId >> customers[i].roomNumber >> customers[i].startDate.day >> customers[i].startDate.month >> customers[i].startDate.year >> customers[i].endDate.day >> customers[i].endDate.month >> customers[i].endDate.year >> customers[i].creditCardNumber;
        }
        inCust.close();
    }
    else
    {
        cout << "Notice: Customer file not found. Creating a new list.\n";
    }

    // Load Reservations
    if (inRes)
    {
        inRes >> reservationCount;
        for (int i = 0; i < reservationCount; i++)
        {
            inRes >> reservations[i].roomNumber >> reservations[i].customerId >> reservations[i].active >> reservations[i].cancelled >> reservations[i].startDate.day >> reservations[i].startDate.month >> reservations[i].startDate.year >> reservations[i].endDate.day >> reservations[i].endDate.month >> reservations[i].endDate.year >> reservations[i].creditCardNumber;
        }
        inRes.close();
    }
    else
    {
        cout << "Notice: Reservation file not found.\n";
    }

    // Load Reviews
    if (inRev)
    {
        inRev >> reviewCount;
        for (int i = 0; i < reviewCount; i++)
        {
            inRev >> reviews[i].customerId >> reviews[i].roomNumber >> reviews[i].dateOfReview.day >> reviews[i].dateOfReview.month >> reviews[i].dateOfReview.year;
            inRev.ignore();
            getline(inRev, reviews[i].reviewContent);
        }
        inRev.close();
    }
    else
    {
        cout << "Notice: Reviews file not found.\n";
    }
}

long long dateToNumber(date d)
{
    return (d.year * 10000) + (d.month * 100) + d.day;
}

bool isValidDate(date s, date e)
{
    // check start date
    if (s.day < 1 || s.day > 31 || s.month < 1 || s.month > 12 || s.year < 2024)
    {
        return false;
    }
    // check end date
    if (e.day < 1 || e.day > 31 || e.month < 1 || e.month > 12 || e.year < 2024)
    {
        return false;
    }
    long long startVal = dateToNumber(s);
    long long endVal = dateToNumber(e);
    if (endVal <= startVal)
    {
        return false;
    }
    return true;
}

bool isValidCreditCard(string card)
{
    if (card.length() < 13 || card.length() > 19)
        return false;

    for (int i = 0; i < card.length(); i++)
    {
        if (!isdigit(card[i]))
            return false;
    }

    int sum = 0;
    bool doubleDigit = false;

    for (int i = card.length() - 1; i >= 0; i--)
    {
        int digit = card[i] - '0';

        if (doubleDigit)
        {
            digit *= 2;

            if (digit > 9)
                digit -= 9;
        }

        sum += digit;
        doubleDigit = !doubleDigit;
    }

    return sum % 10 == 0;
}

bool datesOverlap(date s1, date e1, date s2, date e2)
{
    long long start1 = dateToNumber(s1);
    long long end1 = dateToNumber(e1);
    long long start2 = dateToNumber(s2);
    long long end2 = dateToNumber(e2);

    return start1 < end2 && start2 < end1;
}

bool findRoomByNumber(int roomNumber, int& floorIndex, int& roomIndex)
{
    for (int f = 0; f < FLOORS; f++)
    {
        for (int r = 0; r < ROOMS_PER_FLOOR; r++)
        {
            if (rooms[f][r].roomNumber == roomNumber)
            {
                floorIndex = f;
                roomIndex = r;
                return true;
            }
        }
    }
    return false;
}

bool isRoomAvailableInPeriod(int roomNumber, date startDate, date endDate, int ignoredReservationIndex = -1)
{
    for (int i = 0; i < reservationCount; i++)
    {
        if (i == ignoredReservationIndex)
            continue;

        if (reservations[i].active &&
            reservations[i].roomNumber == roomNumber &&
            datesOverlap(startDate, endDate, reservations[i].startDate, reservations[i].endDate))
        {
            return false;
        }
    }

    return true;
}

void initializeRooms()
{
    string types[] = { "single", "double", "suite" };
    double prices[] = { 500.0, 800.0, 1500.0 };
    for (int i = 0; i < FLOORS; i++)
    {
        for (int j = 0; j < ROOMS_PER_FLOOR; j++)
        {
            rooms[i][j].roomNumber = ((i + 1) * 100) + (j + 1);
            if (rooms[i][j].roomNumber <= 210) // Rooms 101-110 201-210
            {
                rooms[i][j].type = types[0]; // 0 is Single
                rooms[i][j].price = prices[0];
            }
            else if (rooms[i][j].roomNumber <= 410) // Rooms 301-310 401-410
            {
                rooms[i][j].type = types[1]; // 1 is Double
                rooms[i][j].price = prices[1];
            }
            else // Rooms 501-510
            {
                rooms[i][j].type = types[2]; // 2 is Suite
                rooms[i][j].price = prices[2];
            }

            rooms[i][j].hasOffer = (rooms[i][j].type == "Suite");
            if (rooms[i][j].hasOffer)
                rooms[i][j].price *= 0.8;
        }
    }
    admins[0] = { 1, "admin", "1234" };
}

date getTodayDate()
{
    time_t now = time(0);
    tm t;
    localtime_s(&t, &now);

    date today;
    today.day = t.tm_mday;
    today.month = t.tm_mon + 1;
    today.year = t.tm_year + 1900;

    return today;
}

void signUp()
{
    string trialName;
    bool exist;
    do
    {
        exist = false;
        cout << "enter username:\n";
        cin >> trialName;
        for (int i = 0; i < CustomerCount; i++)
        {
            if (customers[i].username == trialName)
            {
                exist = true;
                break;
            }
        }
        if (exist)
            cout << "username exists try another one\n";
    } while (exist);

    customers[CustomerCount].username = trialName;
    cout << "enter your name\n";
    cin.ignore();
    getline(cin, customers[CustomerCount].name);
    string checkEmail;
    bool approved;
    do
    {
        approved = false;
        cout << "enter email\n";
        cin >> checkEmail;
        for (int i = 0; i < checkEmail.size(); i++)
        {
            if (checkEmail[i] == '@')
            {
                approved = true;
                break;
            }
        }
        if (approved)
        {
            customers[CustomerCount].email = checkEmail;
        }
        else
        {
            cout << "invalid email\n";
        }
    } while (!approved);
    cout << "create password:\n";
    cin >> customers[CustomerCount].password;
    customers[CustomerCount].customerId = CustomerCount + 1;
    customers[CustomerCount].roomNumber = -1;
    cout << "Your customer ID is: " << customers[CustomerCount].customerId + ID_OFFSET << endl;
    CustomerCount++;
    cout << "account created successfully\n";
}

void login()
{
    string user, pass;
    bool approved;
    do
    {
        approved = false;
        cout << "enter username and password\n";

        cout << "User: ";
        cin >> user;
        cout << "Pass: ";
        cin >> pass;

        for (int i = 0; i < CustomerCount; i++)
        {
            if (pass == customers[i].password && user == customers[i].username)
            {
                approved = true;
                currentUserIndex = i;
                break;
            }
        }
        if (approved)
            cout << "Logged in successfully" << endl;
        else
            cout << "user and pass don't match\n";
    } while (!approved);
}

void logout()
{
    // تصفير بيانات المستخدم الحالي
    currentUserIndex = -1;
    cout << "You have been logged out successfully." << endl;
    cout << "Thank you for choosing our hotel, we hope to welcome you soon!" << endl;
    // system("pause"); // انتظار ثانيه قبل العوده لشاشة الدخول
}

void checkRoomAvailability()
{
    string reqType;
    date s, e;

    cout << "Enter type (Single/Double/Suite): ";
    cin >> reqType;
    do
    {
        cout << "Start Date (d m y): ";
        cin >> s.day >> s.month >> s.year;

        cout << "End Date (d m y): ";
        cin >> e.day >> e.month >> e.year;

        if (!isValidDate(s, e))
        {
            cout << "\nError: Invalid date or date range! Please try again.\n\n";
        }

    } while (!isValidDate(s, e));

    bool found = false;

    for (int i = 0; i < FLOORS; i++)
    {
        for (int j = 0; j < ROOMS_PER_FLOOR; j++)
        {
            int rn = rooms[i][j].roomNumber;

            if (toLower(rooms[i][j].type) == toLower(reqType) && isRoomAvailableInPeriod(rn, s, e))
            {
                cout << "Available: Room " << rn
                    << " Floor " << i + 1
                    << " Price: " << rooms[i][j].price << endl;

                found = true;
            }
        }
    }

    if (!found)
        cout << "No rooms available for this type in this period." << endl;
}

void ReserveRoom(long long customerId, int roomNumber, date startDate, date endDate, bool ifAdmin)
{
    if (!isValidDate(startDate, endDate))
    {
        cout << "\nError: Invalid date or date range! Please try again.\n\n";
        return;
    }

    long long sVal = dateToNumber(startDate);

    date today = getTodayDate();
    long long todayVal = dateToNumber(today);

    if (sVal < todayVal)
    {
        cout << "Error: You can't reserve room in the past!\n";
        return;
    }

    int customerIndex = -1;

    for (int i = 0; i < CustomerCount; i++)
    {
        if (customers[i].customerId + ID_OFFSET == customerId)
        {
            customerIndex = i;
            break;
        }
    }

    if (customerIndex == -1)
    {
        cout << "Customer not found !!!" << endl;
        return;
    }

    int floorIndex = -1;
    int roomIndex = -1;

    if (!findRoomByNumber(roomNumber, floorIndex, roomIndex))
    {
        cout << "Room does not exist !!!" << endl;
        return;
    }

    if (!isRoomAvailableInPeriod(roomNumber, startDate, endDate))
    {
        cout << "Sorry, this room is already reserved in this period !!!\n";
        return;
    }

    int nights = ((endDate.year - startDate.year) * 365) +
        ((endDate.month - startDate.month) * 30) +
        (endDate.day - startDate.day);

    double roomPrice = rooms[floorIndex][roomIndex].price;
    double roomCost = roomPrice * nights;
    double extrasCost = 0;

    int choice;

    cout << "\nDo you want Sea View? (+300 EGP per night)\n";
    cout << "1. Yes\n2. No\nChoice: ";
    cin >> choice;

    while (choice != 1 && choice != 2)
    {
        cout << "Invalid option, please choose 1 or 2: ";
        cin >> choice;
    }
    if (choice == 1)
    {
        extrasCost += 300 * nights;
    }

    cout << "\nDo you want Meals? (+200 EGP per night)\n";
    cout << "1. Yes\n2. No\nChoice: ";
    cin >> choice;

    while (choice != 1 && choice != 2)
    {
        cout << "Invalid option, please choose 1 or 2: ";
        cin >> choice;
    }
    if (choice == 1)
    {
        extrasCost += 200 * nights;
    }

    cout << "\nDo you want Airport Shuttle? (+400 EGP total)\n";
    cout << "1. Yes\n2. No\nChoice: ";
    cin >> choice;

    while (choice != 1 && choice != 2)
    {
        cout << "Invalid option, please choose 1 or 2: ";
        cin >> choice;
    }
    if (choice == 1)
    {
        extrasCost += 400;
    }

    cout << "\nDo you want Comfort Bed? (+150 EGP per night)\n";
    cout << "1. Yes\n2. No\nChoice: ";
    cin >> choice;

    while (choice != 1 && choice != 2)
    {
        cout << "Invalid option, please choose 1 or 2: ";
        cin >> choice;
    }
    if (choice == 1)
    {
        extrasCost += 150 * nights;
    }

    double totalCost = roomCost + extrasCost;

    cout << "\n--- Reservation Summary ---\n";
    cout << "Room Number: " << roomNumber << endl;
    cout << "Number of Nights: " << nights << endl;
    cout << "Price per Night: " << roomPrice << " EGP\n";
    cout << "Room Cost: " << roomCost << " EGP\n";
    cout << "Extras Cost: " << extrasCost << " EGP\n";
    cout << "Total Cost: " << totalCost << " EGP\n";
    cout << "---------------------------\n\n";

    string creditCardNumber;

    if (!ifAdmin)
    {
        bool validCard;

        do
        {
            cout << "Add Credit Card number to pay " << totalCost << " EGP : ";
            cin >> creditCardNumber;

            validCard = isValidCreditCard(creditCardNumber);

            if (!validCard)
            {
                cout << "Invalid credit card number. Please try again.\n";
            }

        } while (!validCard);
    }

    if (reservationCount >= MAX_RESERVATIONS)
    {
        cout << "Sorry, reservation list is full!\n";
        return;
    }

    reservations[reservationCount].roomNumber = roomNumber;
    reservations[reservationCount].customerId = customers[customerIndex].customerId + ID_OFFSET;
    reservations[reservationCount].startDate = startDate;
    reservations[reservationCount].endDate = endDate;
    reservations[reservationCount].creditCardNumber = creditCardNumber;
    reservations[reservationCount].totalCost = totalCost;
    reservations[reservationCount].active = true;
    reservations[reservationCount].cancelled = false;

    reservationCount++;

    customers[customerIndex].roomNumber = roomNumber;
    customers[customerIndex].startDate = startDate;
    customers[customerIndex].endDate = endDate;
    customers[customerIndex].creditCardNumber = creditCardNumber;

    cout << "Room reserved successfully!\n";
}

void addReview(int customerIdx)
{
    if (customers[customerIdx].roomNumber <= 0) // check lao customer 3ndh reservation aw la2
    {
        cout << "Sorry, you can only add a review if you have an active reservation." << endl;
        return;
    }

    if (reviewCount >= 10)
    {
        cout << "Sorry, We cannot accept more review at the moment." << endl;
        return;
    }
    cout << "\n---- Adding Review for Room Number (" << customers[customerIdx].roomNumber << ") ----\n";

    // rbt data customer automatic B data review//

    reviews[reviewCount].customerId = customers[customerIdx].customerId + ID_OFFSET;
    reviews[reviewCount].roomNumber = customers[customerIdx].roomNumber;

    // bda5l el review content w date of review mn el customer
    cout << "Review content: ";
    cin.ignore();
    getline(cin, reviews[reviewCount].reviewContent);

    reviews[reviewCount].dateOfReview = getTodayDate();

    reviewCount++;
    cout << "Thank you for your review. Your review has been saved!" << endl;
}

void viewRoomReviews()
{
    if (reviewCount == 0)
    {
        cout << "No reviews found.\n";
        return;
    }
    date currentDate = getTodayDate();
    for (int i = 0; i < reviewCount; i++)
    {
        int daysDifference = (currentDate.year - reviews[i].dateOfReview.year) * 365 +
            (currentDate.month - reviews[i].dateOfReview.month) * 30 +
            (currentDate.day - reviews[i].dateOfReview.day);
        if (daysDifference <= 30 && daysDifference >= 0)
        {
            cout << "Customer ID: " << reviews[i].customerId << endl;
            cout << "Room Number: " << reviews[i].roomNumber << endl;
            cout << "Content: " << reviews[i].reviewContent << endl;
            cout << "----------------------\n";
        }
    }
}

void editReservationDates(long long customerId, int roomNumber, date newStart, date newEnd)
{
    int reservationIndex = -1;

    for (int i = 0; i < reservationCount; i++)
    {
        if (reservations[i].active &&
            reservations[i].customerId == customerId &&
            reservations[i].roomNumber == roomNumber)
        {
            reservationIndex = i;
            break;
        }
    }

    if (reservationIndex == -1)
    {
        cout << "Reservation not found for this customer and room !!!\n";
        return;
    }

    if (!isValidDate(newStart, newEnd))
    {
        cout << "\nError: Invalid date or date range! Please try again.\n\n";
        return;
    }

    if (!isRoomAvailableInPeriod(roomNumber, newStart, newEnd, reservationIndex))
    {
        cout << "Error: This room is already booked in the new period !!!\n";
        return;
    }

    reservations[reservationIndex].startDate = newStart;
    reservations[reservationIndex].endDate = newEnd;

    for (int i = 0; i < CustomerCount; i++)
    {
        if (customers[i].customerId + ID_OFFSET == customerId)
        {
            customers[i].startDate = newStart;
            customers[i].endDate = newEnd;
            break;
        }
    }

    // حساب وعرض التكلفة الجديدة بعد تعديل التواريخ
    int floorIndex = -1, roomIndex = -1;
    if (findRoomByNumber(roomNumber, floorIndex, roomIndex))
    {
        int nights = ((newEnd.year - newStart.year) * 365) +
            ((newEnd.month - newStart.month) * 30) +
            (newEnd.day - newStart.day);

        double roomPrice = rooms[floorIndex][roomIndex].price;
        double newTotalCost = roomPrice * nights;
        reservations[reservationIndex].totalCost = newTotalCost;

        cout << "\n--- Updated Reservation Summary ---\n";
        cout << "Room Number: " << roomNumber << endl;
        cout << "New Start Date: " << newStart.day << "/" << newStart.month << "/" << newStart.year << endl;
        cout << "New End Date: " << newEnd.day << "/" << newEnd.month << "/" << newEnd.year << endl;
        cout << "Number of Nights: " << nights << endl;
        cout << "Price per Night: " << roomPrice << " EGP\n";
        cout << "New Total Room Cost: " << newTotalCost << " EGP\n";
        cout << "-----------------------------------\n";
    }

    cout << "Reservation dates updated successfully for Room " << roomNumber << "!\n";
}

void cancelReservation(long long customerId, int roomNumber)
{
    bool isFound = false;

    for (int i = 0; i < reservationCount; i++)
    {
        if (reservations[i].active &&
            reservations[i].customerId == customerId &&
            reservations[i].roomNumber == roomNumber)
        {
            reservations[i].active = false;
            reservations[i].cancelled = true;
            isFound = true;
            break;
        }
    }

    if (!isFound)
    {
        cout << "Reservation not found!" << endl;
        return;
    }

    for (int i = 0; i < CustomerCount; i++)
    {
        if (customers[i].customerId + ID_OFFSET == customerId &&
            customers[i].roomNumber == roomNumber)
        {
            customers[i].roomNumber = 0;
            customers[i].startDate = { 0, 0, 0 };
            customers[i].endDate = { 0, 0, 0 };
            break;
        }
    }

    cout << "Reservation is cancelled successfully" << endl;
}

bool isCompletedReservation(reservation r)
{
    if (r.cancelled)
        return false;

    date today = getTodayDate();

    long long todayVal = dateToNumber(today);
    long long endVal = dateToNumber(r.endDate);

    return endVal < todayVal;
}

void viewReservations()
{
    date today = getTodayDate();
    long long todayVal = dateToNumber(today);

    cout << "\n========== Active Reservations ==========\n";
    bool foundActive = false;

    for (int i = 0; i < reservationCount; i++)
    {
        long long endVal = dateToNumber(reservations[i].endDate);

        if (!reservations[i].cancelled && endVal >= todayVal)
        {
            foundActive = true;

            string customerName = "Unknown";
            for (int j = 0; j < CustomerCount; j++)
            {
                if (customers[j].customerId + ID_OFFSET == reservations[i].customerId)
                {
                    customerName = customers[j].name;
                    break;
                }
            }

            cout << "Customer ID   : " << reservations[i].customerId << endl;
            cout << "Customer Name : " << customerName << endl;
            cout << "Room Number   : " << reservations[i].roomNumber << endl;

            cout << "From          : " << reservations[i].startDate.day << "/"
                << reservations[i].startDate.month << "/"
                << reservations[i].startDate.year << endl;

            cout << "To            : " << reservations[i].endDate.day << "/"
                << reservations[i].endDate.month << "/"
                << reservations[i].endDate.year << endl;

            cout << "Total Cost    : " << reservations[i].totalCost << " EGP\n";
            cout << "Credit Card   : ****" << endl;
            cout << "Status        : Active\n";
            cout << "-----------------------------------------\n";
        }
    }

    if (!foundActive)
        cout << "No active reservations found.\n";


    cout << "\n========== Completed Reservations ==========\n";
    bool foundCompleted = false;

    for (int i = 0; i < reservationCount; i++)
    {
        if (isCompletedReservation(reservations[i]))
        {
            foundCompleted = true;

            string customerName = "Unknown";
            for (int j = 0; j < CustomerCount; j++)
            {
                if (customers[j].customerId + ID_OFFSET == reservations[i].customerId)
                {
                    customerName = customers[j].name;
                    break;
                }
            }

            cout << "Customer ID   : " << reservations[i].customerId << endl;
            cout << "Customer Name : " << customerName << endl;
            cout << "Room Number   : " << reservations[i].roomNumber << endl;

            cout << "From          : " << reservations[i].startDate.day << "/"
                << reservations[i].startDate.month << "/"
                << reservations[i].startDate.year << endl;

            cout << "To            : " << reservations[i].endDate.day << "/"
                << reservations[i].endDate.month << "/"
                << reservations[i].endDate.year << endl;

            cout << "Total Cost    : " << reservations[i].totalCost << " EGP\n";
            cout << "Status        : Completed\n";
            cout << "-----------------------------------------\n";
        }
    }

    if (!foundCompleted)
        cout << "No completed reservations found.\n";


    cout << "\n========== Cancelled Reservations ==========\n";
    bool foundCancelled = false;

    for (int i = 0; i < reservationCount; i++)
    {
        if (reservations[i].cancelled)
        {
            foundCancelled = true;

            string customerName = "Unknown";
            for (int j = 0; j < CustomerCount; j++)
            {
                if (customers[j].customerId + ID_OFFSET == reservations[i].customerId)
                {
                    customerName = customers[j].name;
                    break;
                }
            }

            cout << "Customer ID   : " << reservations[i].customerId << endl;
            cout << "Customer Name : " << customerName << endl;
            cout << "Room Number   : " << reservations[i].roomNumber << endl;

            cout << "From          : " << reservations[i].startDate.day << "/"
                << reservations[i].startDate.month << "/"
                << reservations[i].startDate.year << endl;

            cout << "To            : " << reservations[i].endDate.day << "/"
                << reservations[i].endDate.month << "/"
                << reservations[i].endDate.year << endl;

            cout << "Total Cost    : " << reservations[i].totalCost << " EGP\n";
            cout << "Status        : Cancelled\n";
            cout << "-----------------------------------------\n";
        }
    }

    if (!foundCancelled)
        cout << "No cancelled reservations found.\n";
}

void viewMyReservation(int customerIdx)
{
    long long myId = customers[customerIdx].customerId + ID_OFFSET;

    date today = getTodayDate();
    long long todayVal = dateToNumber(today);

    cout << "\n========================================\n";
    cout << "         MY CURRENT RESERVATIONS         \n";
    cout << "========================================\n";

    bool hasActive = false;

    for (int i = 0; i < reservationCount; i++)
    {
        long long endVal = dateToNumber(reservations[i].endDate);

        if (reservations[i].customerId == myId &&
            !reservations[i].cancelled &&
            endVal >= todayVal)
        {
            hasActive = true;

            cout << "Room Number : " << reservations[i].roomNumber << "\n";

            cout << "Check-in    : " << reservations[i].startDate.day << "/"
                << reservations[i].startDate.month << "/"
                << reservations[i].startDate.year << "\n";

            cout << "Check-out   : " << reservations[i].endDate.day << "/"
                << reservations[i].endDate.month << "/"
                << reservations[i].endDate.year << "\n";

            cout << "Total Cost  : " << reservations[i].totalCost << " EGP\n";
            cout << "Status      : Active\n";
            cout << "----------------------------------------\n";
        }
    }

    if (!hasActive)
        cout << "You have no current reservations.\n";


    cout << "\n========================================\n";
    cout << "         MY COMPLETED RESERVATIONS       \n";
    cout << "========================================\n";

    bool hasCompleted = false;

    for (int i = 0; i < reservationCount; i++)
    {
        if (reservations[i].customerId == myId &&
            isCompletedReservation(reservations[i]))
        {
            hasCompleted = true;

            cout << "Room Number : " << reservations[i].roomNumber << "\n";

            cout << "Check-in    : " << reservations[i].startDate.day << "/"
                << reservations[i].startDate.month << "/"
                << reservations[i].startDate.year << "\n";

            cout << "Check-out   : " << reservations[i].endDate.day << "/"
                << reservations[i].endDate.month << "/"
                << reservations[i].endDate.year << "\n";

            cout << "Total Cost  : " << reservations[i].totalCost << " EGP\n";
            cout << "Status      : Completed\n";
            cout << "----------------------------------------\n";
        }
    }

    if (!hasCompleted)
        cout << "No completed reservations found.\n";


    cout << "\n========================================\n";
    cout << "         MY CANCELLED RESERVATIONS       \n";
    cout << "========================================\n";

    bool hasCancelled = false;

    for (int i = 0; i < reservationCount; i++)
    {
        if (reservations[i].customerId == myId &&
            reservations[i].cancelled)
        {
            hasCancelled = true;

            cout << "Room Number : " << reservations[i].roomNumber << "\n";

            cout << "Check-in    : " << reservations[i].startDate.day << "/"
                << reservations[i].startDate.month << "/"
                << reservations[i].startDate.year << "\n";

            cout << "Check-out   : " << reservations[i].endDate.day << "/"
                << reservations[i].endDate.month << "/"
                << reservations[i].endDate.year << "\n";

            cout << "Total Cost  : " << reservations[i].totalCost << " EGP\n";
            cout << "Status      : Cancelled\n";
            cout << "----------------------------------------\n";
        }
    }

    if (!hasCancelled)
        cout << "No cancelled reservations found.\n";
}

int main()
{
    initializeRooms();
    loadData();

    int mainChoice;
    while (true)
    {
        cout << "\n1. Customer\n2. Admin\n3. Exit\nChoice: ";
        cin >> mainChoice;

        if (mainChoice == 1) // Customer Section
        {
            cout << "1. SignUp\n2. Login\nChoice: ";
            int c;
            cin >> c;

            if (c == 1)
            {
                signUp();
            }
            else if (c == 2)
            {
                login();
                while (true)
                {
                    cout << "\n1. Check Availability\n2. Add Review\n3. Make Reservation\n4. My Reservation\n5. Logout\nChoice: ";
                    int uc;
                    cin >> uc;

                    if (uc == 1)
                        checkRoomAvailability();
                    else if (uc == 2)
                        addReview(currentUserIndex);
                    else if (uc == 3)
                    {
                        int rn;
                        date s, e;
                        cout << "Room Number: ";
                        cin >> rn;
                        do
                        {
                            cout << "Start Date (d m y): ";
                            cin >> s.day >> s.month >> s.year;
                            cout << "End Date (d m y): ";
                            cin >> e.day >> e.month >> e.year;
                            if (!isValidDate(s, e))
                            {
                                cout << "\nError: Invalid date or date range! Please try again.\n\n";
                            }
                        } while (!isValidDate(s, e));

                        ReserveRoom(customers[currentUserIndex].customerId + ID_OFFSET, rn, s, e, false);
                    }
                    else if (uc == 4)
                    {
                        viewMyReservation(currentUserIndex);
                    }
                    else if (uc == 5)
                    {
                        logout();
                        break;
                    }
                }
            }
            else
                cout << "Invalid option\n";
        }
        else if (mainChoice == 2) // Admin Section
        {
            string u, p;
            cout << "Admin User: ";
            cin >> u;
            cout << "Admin Pass: ";
            cin >> p;

            if (u == admins[0].username && p == admins[0].password)
            {
                while (true)
                {
                    cout << "\n1. View Reviews\n2. Add Reservation\n3. Cancel Reservation\n4. Edit Reservation\n5. View Reservations\n6. Logout\nChoice: ";
                    int ac;
                    cin >> ac;

                    if (ac == 1)
                    {
                        viewRoomReviews();
                    }
                    else if (ac == 2)
                    {
                        long long cId;
                        int rn;
                        date s, e;
                        cout << "Customer ID to reserve: ";
                        cin >> cId;
                        cout << "Room Num: ";
                        cin >> rn;
                        do
                        {
                            cout << "Start (d m y): ";
                            cin >> s.day >> s.month >> s.year;
                            cout << "End (d m y): ";
                            cin >> e.day >> e.month >> e.year;

                            if (!isValidDate(s, e))
                            {
                                cout << "\nError: Invalid date or date range! Please try again.\n\n";
                            }
                        } while (!isValidDate(s, e));
                        ReserveRoom(cId, rn, s, e, true);
                    }
                    else if (ac == 3)
                    {
                        long long cId;
                        int rn;

                        cout << "Customer ID to cancel: ";
                        cin >> cId;

                        cout << "Room Number to cancel: ";
                        cin >> rn;

                        cancelReservation(cId, rn);
                    }
                    else if (ac == 4)
                    {
                        long long cId;
                        int rn;
                        date newS, newE;

                        cout << "Confirm Customer ID: ";
                        cin >> cId;

                        cout << "Room Number: ";
                        cin >> rn;

                        do
                        {
                            cout << "New Start (d m y): ";
                            cin >> newS.day >> newS.month >> newS.year;

                            cout << "New End (d m y): ";
                            cin >> newE.day >> newE.month >> newE.year;

                            if (!isValidDate(newS, newE))
                            {
                                cout << "\nError: Invalid date or date range! Please try again.\n\n";
                            }

                        } while (!isValidDate(newS, newE));

                        editReservationDates(cId, rn, newS, newE);
                    }
                    else if (ac == 5)
                    {
                        viewReservations();
                    }
                    else if (ac == 6)
                        break;
                    else
                        cout << "Invalid admin credentials!\n"
                        "please try again\n";
                }
            }
            else
            {
                cout << "user and pass dont match please try again\n";
            }
        }

        else if (mainChoice == 3)
        {
            saveData();
            break;
        }
        else
        {
            cout << "Invalid option\n";
        }
    }
    return 0;
}
