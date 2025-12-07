#include <stdio.h>
#include <stdlib.h>
#include <string.h>

/* =========================== FILE NAME =========================== */

#define STUD_FILE "students.dat"

/* =========================== STRUCTURES ========================== */

typedef struct {
    char  id[20];
    char  name[50];
    char  branch[20];
    char  section[10];
    char  phone[15];
    char  password[20];
    float cgpa;

    /* Marks */
    int   cppMarks;      /* C++ */
    int   daaMarks;      /* DAA */
    int   csMarks;       /* Coding Skills */

    /* Attendance: sequence of 'P' and 'A' (each character = 1 day) */
    char  cppAtt[101];   /* C++ attendance record */
    char  daaAtt[101];   /* DAA attendance record */
    char  csAtt[101];    /* Coding Skills attendance record */
} Student;

/* =========================== UTILITY ============================= */

void clearInput() {
    int c;
    while ((c = getchar()) != '\n' && c != EOF) {
        /* discard */
    }
}

/* =========================== LOGIN =============================== */

int adminLogin() {
    char username[20];
    char password[20];

    printf("\n========== ADMIN LOGIN ==========\n");
    printf("Username: ");
    scanf("%19s", username);

    printf("Password: ");
    scanf("%19s", password);

    if (!strcmp(username, "admin") && !strcmp(password, "admin123")) {
        return 1;
    } else {
        printf("\n[!] Invalid admin credentials.\n");
        return 0;
    }
}

int staffLogin(char staffId[]) {
    char username[20];
    char password[20];

    printf("\n========== STAFF LOGIN ==========\n");
    printf("Staff ID: ");
    scanf("%19s", username);

    printf("Password: ");
    scanf("%19s", password);

    /* Demo staff login: staff / staff123 */
    if (!strcmp(username, "staff") && !strcmp(password, "staff123")) {
        strcpy(staffId, username);
        return 1;
    } else {
        printf("\n[!] Invalid staff credentials.\n");
        return 0;
    }
}

int studentLogin(char studentId[]) {
    FILE   *fp;
    Student s;
    char    id[20];
    char    pass[20];
    int     found = 0;

    fp = fopen(STUD_FILE, "rb");
    if (!fp) {
        printf("\n[!] No student database file found.\n");
        return 0;
    }

    printf("\n========== STUDENT LOGIN ==========\n");
    printf("ID: ");
    scanf("%19s", id);

    printf("Password: ");
    scanf("%19s", pass);

    while (fread(&s, sizeof(Student), 1, fp)) {
        if (!strcmp(s.id, id) && !strcmp(s.password, pass)) {
            found = 1;
            strcpy(studentId, id);
            break;
        }
    }

    fclose(fp);

    if (!found) {
        printf("\n[!] Invalid student ID or password.\n");
        return 0;
    }

    return 1;
}

/* =========================== ADMIN CRUD ========================== */

void addStudent() {
    FILE   *fp;
    Student s;

    fp = fopen(STUD_FILE, "ab");
    if (!fp) {
        printf("\n[!] Error opening student file.\n");
        return;
    }

    printf("\n========== ADD STUDENT (ADMIN) ==========\n");
    printf("ID: ");
    scanf("%19s", s.id);
    clearInput();

    printf("Name: ");
    fgets(s.name, sizeof(s.name), stdin);
    s.name[strcspn(s.name, "\n")] = '\0';

    printf("Branch: ");
    fgets(s.branch, sizeof(s.branch), stdin);
    s.branch[strcspn(s.branch, "\n")] = '\0';

    printf("Section: ");
    fgets(s.section, sizeof(s.section), stdin);
    s.section[strcspn(s.section, "\n")] = '\0';

    printf("CGPA: ");
    scanf("%f", &s.cgpa);
    clearInput();

    printf("Phone: ");
    fgets(s.phone, sizeof(s.phone), stdin);
    s.phone[strcspn(s.phone, "\n")] = '\0';

    /* Default password: id@srm */
    sprintf(s.password, "%s@srm", s.id);

    /* Initialize marks as -1 meaning "not entered yet" */
    s.cppMarks = -1;
    s.daaMarks = -1;
    s.csMarks  = -1;

    /* Initialize attendance strings as empty */
    s.cppAtt[0] = '\0';
    s.daaAtt[0] = '\0';
    s.csAtt[0]  = '\0';

    fwrite(&s, sizeof(Student), 1, fp);
    fclose(fp);

    printf("\n[+] Student added successfully.\n");
    printf("    Default Password: %s\n", s.password);
}

void viewStudents() {
    FILE   *fp;
    Student s;
    int     count = 0;

    fp = fopen(STUD_FILE, "rb");
    if (!fp) {
        printf("\n[!] No student records found.\n");
        return;
    }

    printf("\n========== ALL STUDENT RECORDS ==========\n");

    while (fread(&s, sizeof(Student), 1, fp)) {
        printf("\n----------------------------------------\n");
        printf("ID      : %s\n", s.id);
        printf("Name    : %s\n", s.name);
        printf("Branch  : %s\n", s.branch);
        printf("Section : %s\n", s.section);
        printf("CGPA    : %.2f\n", s.cgpa);
        printf("Phone   : %s\n", s.phone);

        /* Marks */
        if (s.cppMarks >= 0)
            printf("C++ Marks        : %d\n", s.cppMarks);
        else
            printf("C++ Marks        : Not entered\n");

        if (s.daaMarks >= 0)
            printf("DAA Marks        : %d\n", s.daaMarks);
        else
            printf("DAA Marks        : Not entered\n");

        if (s.csMarks >= 0)
            printf("Coding Skills    : %d\n", s.csMarks);
        else
            printf("Coding Skills    : Not entered\n");

        /* Attendance */
        if (strlen(s.cppAtt) > 0)
            printf("C++ Attendance   : %s\n", s.cppAtt);
        else
            printf("C++ Attendance   : No records\n");

        if (strlen(s.daaAtt) > 0)
            printf("DAA Attendance   : %s\n", s.daaAtt);
        else
            printf("DAA Attendance   : No records\n");

        if (strlen(s.csAtt) > 0)
            printf("Coding Skills Att: %s\n", s.csAtt);
        else
            printf("Coding Skills Att: No records\n");

        count++;
    }

    if (count == 0) {
        printf("\n[!] No students in database.\n");
    }

    fclose(fp);
}

void searchStudent() {
    FILE   *fp;
    Student s;
    char    id[20];
    int     found = 0;

    printf("\n========== SEARCH STUDENT ==========\n");
    printf("Enter Student ID: ");
    scanf("%19s", id);

    fp = fopen(STUD_FILE, "rb");
    if (!fp) {
        printf("\n[!] No student records found.\n");
        return;
    }

    while (fread(&s, sizeof(Student), 1, fp)) {
        if (!strcmp(s.id, id)) {
            printf("\n[+] Student Found:\n");
            printf("----------------------------------------\n");
            printf("ID      : %s\n", s.id);
            printf("Name    : %s\n", s.name);
            printf("Branch  : %s\n", s.branch);
            printf("Section : %s\n", s.section);
            printf("CGPA    : %.2f\n", s.cgpa);
            printf("Phone   : %s\n", s.phone);

            if (s.cppMarks >= 0)
                printf("C++ Marks        : %d\n", s.cppMarks);
            else
                printf("C++ Marks        : Not entered\n");

            if (s.daaMarks >= 0)
                printf("DAA Marks        : %d\n", s.daaMarks);
            else
                printf("DAA Marks        : Not entered\n");

            if (s.csMarks >= 0)
                printf("Coding Skills    : %d\n", s.csMarks);
            else
                printf("Coding Skills    : Not entered\n");

            if (strlen(s.cppAtt) > 0)
                printf("C++ Attendance   : %s\n", s.cppAtt);
            else
                printf("C++ Attendance   : No records\n");

            if (strlen(s.daaAtt) > 0)
                printf("DAA Attendance   : %s\n", s.daaAtt);
            else
                printf("DAA Attendance   : No records\n");

            if (strlen(s.csAtt) > 0)
                printf("Coding Skills Att: %s\n", s.csAtt);
            else
                printf("Coding Skills Att: No records\n");

            found = 1;
            break;
        }
    }

    fclose(fp);

    if (!found) {
        printf("\n[!] Student with ID %s not found.\n", id);
    }
}

void updateStudent() {
    FILE   *fp;
    Student s;
    char    id[20];
    int     found = 0;

    printf("\n========== UPDATE STUDENT (ADMIN) ==========\n");
    printf("Enter Student ID: ");
    scanf("%19s", id);

    fp = fopen(STUD_FILE, "rb+");
    if (!fp) {
        printf("\n[!] No student records found.\n");
        return;
    }

    while (fread(&s, sizeof(Student), 1, fp)) {
        if (!strcmp(s.id, id)) {
            found = 1;
            clearInput();

            printf("New Name    : ");
            fgets(s.name, sizeof(s.name), stdin);
            s.name[strcspn(s.name, "\n")] = '\0';

            printf("New Branch  : ");
            fgets(s.branch, sizeof(s.branch), stdin);
            s.branch[strcspn(s.branch, "\n")] = '\0';

            printf("New Section : ");
            fgets(s.section, sizeof(s.section), stdin);
            s.section[strcspn(s.section, "\n")] = '\0';

            printf("New CGPA    : ");
            scanf("%f", &s.cgpa);
            clearInput();

            printf("New Phone   : ");
            fgets(s.phone, sizeof(s.phone), stdin);
            s.phone[strcspn(s.phone, "\n")] = '\0';

            fseek(fp, - (long)sizeof(Student), SEEK_CUR);
            fwrite(&s, sizeof(Student), 1, fp);

            printf("\n[+] Student updated successfully.\n");
            fclose(fp);
            return;
        }
    }

    fclose(fp);

    if (!found) {
        printf("\n[!] Student with ID %s not found.\n", id);
    }
}

void deleteStudent() {
    FILE   *fp, *temp;
    Student s;
    char    id[20];
    int     found = 0;

    printf("\n========== DELETE STUDENT (ADMIN) ==========\n");
    printf("Enter Student ID to delete: ");
    scanf("%19s", id);

    fp   = fopen(STUD_FILE, "rb");
    temp = fopen("temp.dat", "wb");

    if (!fp || !temp) {
        printf("\n[!] File error while deleting.\n");
        if (fp) fclose(fp);
        if (temp) fclose(temp);
        return;
    }

    while (fread(&s, sizeof(Student), 1, fp)) {
        if (!strcmp(s.id, id)) {
            found = 1;
            /* skip writing: this will delete */
        } else {
            fwrite(&s, sizeof(Student), 1, temp);
        }
    }

    fclose(fp);
    fclose(temp);

    remove(STUD_FILE);
    rename("temp.dat", STUD_FILE);

    if (found) {
        printf("\n[+] Student with ID %s deleted.\n", id);
    } else {
        printf("\n[!] Student with ID %s not found.\n", id);
    }
}

/* =========================== STUDENT VIEW ========================= */

void viewMyDetails(char sid[]) {
    FILE   *fp;
    Student s;
    int     found = 0;

    fp = fopen(STUD_FILE, "rb");
    if (!fp) {
        printf("\n[!] No student records found.\n");
        return;
    }

    while (fread(&s, sizeof(Student), 1, fp)) {
        if (!strcmp(s.id, sid)) {
            printf("\n========== MY DETAILS ==========\n");
            printf("ID      : %s\n", s.id);
            printf("Name    : %s\n", s.name);
            printf("Branch  : %s\n", s.branch);
            printf("Section : %s\n", s.section);
            printf("CGPA    : %.2f\n", s.cgpa);
            printf("Phone   : %s\n", s.phone);

            if (s.cppMarks >= 0)
                printf("C++ Marks        : %d\n", s.cppMarks);
            else
                printf("C++ Marks        : Not entered\n");

            if (s.daaMarks >= 0)
                printf("DAA Marks        : %d\n", s.daaMarks);
            else
                printf("DAA Marks        : Not entered\n");

            if (s.csMarks >= 0)
                printf("Coding Skills    : %d\n", s.csMarks);
            else
                printf("Coding Skills    : Not entered\n");

            if (strlen(s.cppAtt) > 0)
                printf("C++ Attendance   : %s\n", s.cppAtt);
            else
                printf("C++ Attendance   : No records\n");

            if (strlen(s.daaAtt) > 0)
                printf("DAA Attendance   : %s\n", s.daaAtt);
            else
                printf("DAA Attendance   : No records\n");

            if (strlen(s.csAtt) > 0)
                printf("Coding Skills Att: %s\n", s.csAtt);
            else
                printf("Coding Skills Att: No records\n");

            found = 1;
            break;
        }
    }

    fclose(fp);

    if (!found) {
        printf("\n[!] Your record was not found.\n");
    }
}

/* ======================== STAFF: ENTER MARKS ====================== */

void enterMarks(char staffId[]) {
    (void)staffId;  /* currently unused, kept for future use */

    FILE   *fp;
    Student s;
    char    id[20];
    int     found = 0;

    printf("\n========== ENTER MARKS (STAFF) ==========\n");
    printf("Enter Student ID: ");
    scanf("%19s", id);

    fp = fopen(STUD_FILE, "rb+");
    if (!fp) {
        printf("\n[!] No student records found.\n");
        return;
    }

    while (fread(&s, sizeof(Student), 1, fp)) {
        if (!strcmp(s.id, id)) {
            found = 1;

            printf("\nEnter marks for student %s (%s)\n", s.id, s.name);

            printf("C++ Marks           : ");
            scanf("%d", &s.cppMarks);

            printf("DAA Marks           : ");
            scanf("%d", &s.daaMarks);

            printf("Coding Skills Marks : ");
            scanf("%d", &s.csMarks);

            fseek(fp, - (long)sizeof(Student), SEEK_CUR);
            fwrite(&s, sizeof(Student), 1, fp);

            printf("\n[+] Marks updated successfully.\n");
            fclose(fp);
            return;
        }
    }

    fclose(fp);

    if (!found) {
        printf("\n[!] Student with ID %s not found.\n", id);
    }
}

/* ====================== STAFF: ENTER ATTENDANCE =================== */

void appendAttendanceChar(char *attString, char status) {
    int len = (int)strlen(attString);

    if (len >= 100) {
        printf("\n[!] Attendance record is full (max 100 days).\n");
        return;
    }

    attString[len]     = status;
    attString[len + 1] = '\0';
}

void enterAttendance(char staffId[]) {
    (void)staffId;  /* not used now */

    FILE   *fp;
    Student s;
    char    id[20];
    int     found = 0;
    int     choice;
    char    statusChar;

    printf("\n========== ENTER ATTENDANCE (STAFF) ==========\n");
    printf("Enter Student ID: ");
    scanf("%19s", id);

    fp = fopen(STUD_FILE, "rb+");
    if (!fp) {
        printf("\n[!] No student records found.\n");
        return;
    }

    while (fread(&s, sizeof(Student), 1, fp)) {
        if (!strcmp(s.id, id)) {
            found = 1;

            printf("\nStudent: %s (%s)\n", s.id, s.name);
            printf("Existing Attendance:\n");
            printf("  C++            : %s\n", strlen(s.cppAtt) ? s.cppAtt : "No records");
            printf("  DAA            : %s\n", strlen(s.daaAtt) ? s.daaAtt : "No records");
            printf("  Coding Skills  : %s\n", strlen(s.csAtt)  ? s.csAtt  : "No records");

            printf("\nWhich subject attendance do you want to mark for TODAY?\n");
            printf("1. C++\n");
            printf("2. DAA\n");
            printf("3. Coding Skills\n");
            printf("4. All three subjects\n");
            printf("Enter choice: ");
            scanf("%d", &choice);

            printf("\nMark as (P = Present, A = Absent): ");
            clearInput();
            scanf("%c", &statusChar);

            if (statusChar == 'p' || statusChar == 'P') {
                statusChar = 'P';
            } else if (statusChar == 'a' || statusChar == 'A') {
                statusChar = 'A';
            } else {
                printf("\n[!] Invalid status. Use P or A only.\n");
                fclose(fp);
                return;
            }

            if (choice == 1 || choice == 4) {
                appendAttendanceChar(s.cppAtt, statusChar);
            }
            if (choice == 2 || choice == 4) {
                appendAttendanceChar(s.daaAtt, statusChar);
            }
            if (choice == 3 || choice == 4) {
                appendAttendanceChar(s.csAtt, statusChar);
            }

            fseek(fp, - (long)sizeof(Student), SEEK_CUR);
            fwrite(&s, sizeof(Student), 1, fp);

            printf("\n[+] Attendance updated successfully.\n");

            fclose(fp);
            return;
        }
    }

    fclose(fp);

    if (!found) {
        printf("\n[!] Student with ID %s not found.\n", id);
    }
}

/* ============================== MENUS ============================ */

void adminMenu() {
    int choice;

    while (1) {
        printf("\n========== ADMIN MENU ==========\n");
        printf("1. Add Student\n");
        printf("2. View All Students\n");
        printf("3. Search Student\n");
        printf("4. Update Student\n");
        printf("5. Delete Student\n");
        printf("0. Logout\n");
        printf("Enter your choice: ");
        scanf("%d", &choice);

        switch (choice) {
            case 1: addStudent();    break;
            case 2: viewStudents();  break;
            case 3: searchStudent(); break;
            case 4: updateStudent(); break;
            case 5: deleteStudent(); break;
            case 0: return;
            default:
                printf("\n[!] Invalid choice. Try again.\n");
        }
    }
}

void staffMenu(char staffId[]) {
    int choice;

    while (1) {
        printf("\n========== STAFF MENU ==========\n");
        printf("1. View All Students\n");
        printf("2. Search Student\n");
        printf("3. Enter Marks (C++, DAA, Coding Skills)\n");
        printf("4. Enter Attendance (P/A for subjects)\n");
        printf("0. Logout\n");
        printf("Enter your choice: ");
        scanf("%d", &choice);

        switch (choice) {
            case 1: viewStudents();        break;
            case 2: searchStudent();       break;
            case 3: enterMarks(staffId);   break;
            case 4: enterAttendance(staffId); break;
            case 0: return;
            default:
                printf("\n[!] Invalid choice. Try again.\n");
        }
    }
}

void studentMenu(char studentId[]) {
    int choice;

    while (1) {
        printf("\n========== STUDENT MENU ==========\n");
        printf("1. View My Details (Marks + Attendance)\n");
        printf("0. Logout\n");
        printf("Enter your choice: ");
        scanf("%d", &choice);

        if (choice == 1) {
            viewMyDetails(studentId);
        } else if (choice == 0) {
            return;
        } else {
            printf("\n[!] Invalid choice. Try again.\n");
        }
    }
}

/* ============================== MAIN ============================= */

int main() {
    int  choice;
    char staffId[20];
    char studentId[20];

    while (1) {
        printf("\n========== MAIN MENU ==========\n");
        printf("1. Admin Login\n");
        printf("2. Staff Login\n");
        printf("3. Student Login\n");
        printf("0. Exit\n");
        printf("Enter your choice: ");
        scanf("%d", &choice);

        if (choice == 1) {
            if (adminLogin()) {
                adminMenu();
            }
        } else if (choice == 2) {
            if (staffLogin(staffId)) {
                staffMenu(staffId);
            }
        } else if (choice == 3) {
            if (studentLogin(studentId)) {
                studentMenu(studentId);
            }
        } else if (choice == 0) {
            printf("\nExiting program. Goodbye!\n");
            break;
        } else {
            printf("\n[!] Invalid choice. Try again.\n");
        }
    }

    return 0;
}