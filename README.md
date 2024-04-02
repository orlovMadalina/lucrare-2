
#include <iostream>
#include <string>
#include <list>
#include <fstream>
#include <sstream>
#include <ctime>

# define SAVE "University.txt"

using namespace std;

void Pause(){
  cin.ignore();
  cout << "\nPress Enter to continue...";
  getchar();
}

enum StudyField {
  MECHANICAL_ENGINEERING,
  SOFTWARE_ENGINEERING,
  FOOD_TECHNOLOGY,
  URBANISM_ARCHITECTURE,
  VETERINARY_MEDICINE
};

class Student {
private:
  string FirstName, LastName, Email;
  string EnrollmentDate, DateOfBirth; // It's pain to deal with c date type
  bool IsGraduated;

public:
  Student(string first, string last, string email, string enrollment, string dob)
      : FirstName(first), LastName(last), Email(email), EnrollmentDate(enrollment), DateOfBirth(dob), IsGraduated(false) {}

  Student(string first, string last, string email, string enrollment, string dob,bool grad)
      : FirstName(first), LastName(last), Email(email), EnrollmentDate(enrollment), DateOfBirth(dob), IsGraduated(grad) {}

  void Print_Info() { cout << '\n'<< FirstName << '|' << LastName << '|' << Email << '|' << EnrollmentDate << '|'
    << DateOfBirth;}

  const string& GetFirstName(){return FirstName;}
  const string& GetLastName(){return LastName;}
  const string& GetEmail() { return Email; }
  const string& GetEnrollmentDate() { return EnrollmentDate; }
  const string& GetDateOfBirth() { return DateOfBirth; }
  const bool GetIsGraduated() { return IsGraduated; }

  void Graduate() { IsGraduated = true; }
};

class Faculty {
private:
  string Name, Abreviation;
  list<Student> students;
  StudyField studyfield;

public:
  Faculty(string name, string abbreviation, StudyField field)
      : Name(name), Abreviation(abbreviation), studyfield(field) {}

  const string GetName() { return Name; }

  const string GetAbreviation() { return Abreviation; }

  void AddStudent(const Student& student) {
    students.push_back(student);
  }

  list<Student>& GetStudents() { return students; }

  StudyField GetStudyField() { return studyfield; }

  void DisplayStudents() {
    cout << "Students enrolled in " << Name << " faculty:\n";
    for ( auto& student : students) {
      student.Print_Info();
    }
    cout << endl;
  Pause();}

  void DisplayGraduates() {
    cout << "Graduates from " << Name << " faculty:\n";
    for ( auto& student : students) {
      if (student.GetIsGraduated()) {
        student.Print_Info();
      }
    }
    cout << endl;
  Pause();}

  bool CheckStudent(const string& email) {
    for ( auto& student : students) {
      if (student.GetEmail() == email) {
        return true;
      }
    }
    return false;
  }

  void GraduateStudent(const string& email) {
    for (auto& student : students) {
      if (student.GetEmail() == email) {
        student.Graduate();
        cout << student.GetFirstName() << ' ' << student.GetLastName() << " graduated from " << Name << " faculty.\n";
        return;
      }
    }
    cout << "Student with email '" << email << "' not found in " << Name << " faculty.\n";
  }
};

class University {
  //private: list<Faculty> faculties;

  public:list<Faculty> faculties;
    void CreateFaculty(string name, string abbreviation, StudyField field) {
      faculties.push_back(Faculty(name, abbreviation, field));
    }

    Faculty *FindFacultyByStudentEmail(const string &email) {
      for (auto &faculty : faculties) {
        if (faculty.CheckStudent(email)) {
          return &faculty;
        }
      }
      return nullptr;
    }

    void DisplayFaculties() {
      cout << "University Faculties:\n";
      for (auto &faculty : faculties) {
        cout << "- " << faculty.GetAbreviation() << " Faculty: " << faculty.GetName() << endl;
      }
      Pause();
      cout << endl;
    }

    void DisplayFacultiesByField(StudyField field) {
      cout << "Faculties in " << field << " field:\n";
      for (auto &faculty : faculties) {
        if (faculty.GetStudyField() == field) {
          cout << "- " << faculty.GetAbreviation() << " Faculty: " << faculty.GetName() << endl;
        }
      }
      cout << endl;
    }

    void DisplayStudentsInFaculty(const string &abbreviation) {
      for (auto &faculty : faculties) {
        if (faculty.GetAbreviation() == abbreviation) {
          faculty.DisplayStudents();
          return;
        }
      }
      cout << "Faculty with abbreviation '" << abbreviation << "' not found!\n";
    }

    void DisplayGraduatesInFaculty(const string &abbreviation) {
      for (auto &faculty : faculties) {
        if (faculty.GetAbreviation() == abbreviation) {
          faculty.DisplayGraduates();
          return;
        }
      }
      cout << "Faculty with abbreviation '" << abbreviation << "' not found!\n";
    }

    list<Faculty> &GetFaculties() {
      return faculties;
    }
};

class FileManager {
  public:
    static void SaveUniversityData(const string &filename, University &university) {
      ofstream outfile(filename);
      if (outfile.is_open()) {
        for (auto &faculty : university.GetFaculties()) {
          outfile << faculty.GetName() << " " << faculty.GetAbreviation() << " " << faculty.GetStudyField() << endl;
          for (auto &student : faculty.GetStudents()) {
            outfile << student.GetFirstName() << " " << student.GetLastName() << " " << student.GetEmail()
            << " " << student.GetEnrollmentDate() << " " << student.GetDateOfBirth() << " " << student.GetIsGraduated() << endl;
          }
          outfile << endl; // Separate faculties with an empty line
        }
        cout << "University data saved successfully!\n";
      } else {
        cout << "Unable to open file for saving!\n";
      }
      outfile.close();
    }

    static void LoadUniversityData(const string &filename, University &university) {
      ifstream infile(filename);
      if (infile.is_open()) {
        string line;
        while (getline(infile, line)) {
          string name, abbreviation;
          int field;
          stringstream ss(line);
          ss >> name >> abbreviation >> field;
          university.CreateFaculty(name, abbreviation, static_cast<StudyField>(field));

          while (getline(infile, line) && !line.empty()) {
            stringstream ss2(line);
            string first, last, email,enroll,birth;
            bool isGraduated = false;
            ss2 >> first >> last >> email >> enroll >> birth >> isGraduated;
            university.GetFaculties().back().AddStudent(Student(first, last, email, enroll, birth,isGraduated));
          }
        }
        cout << "University data loaded successfully!\n";
      } else {
        cout << "Unable to open file for loading!\n";
      }
      infile.close();
    }
};

enum Level{
  ERROR,
  WARNING,
  INFO,
  DEBUG
};

class Log {
private:
    Level logLevel;

public:
    // Constructor to set the log level
    Log(Level level) : logLevel(level) {}

    // Log message based on log level
    void createLogFile() {
        std::ofstream createFile("log.txt");
        if (!createFile.is_open()) {
            std::cerr << "Error: Unable to create log file!" << std::endl;
        } else {
            std::cerr << "Log file created successfully!" << std::endl;
        }
    }

    // Log message based on log level
    void LogMessage(Level level, const std::string &message) {
        if (logLevel >= level) {
            std::ofstream logfile("log.txt", std::ios::app);
            if (!logfile.is_open()) {
                std::cerr << "Error: Unable to open log file for writing!" << std::endl;
                // Create the log file if it doesn't exist
                createLogFile();
                // Try opening the log file again
                logfile.open("log.txt", std::ios::app);
                if (!logfile.is_open()) {
                    std::cerr << "Error: Unable to open log file even after creation!" << std::endl;
                    return;
                }
            }

            std::string levelString;
            switch (level) {
                case Level::ERROR:
                    levelString = "ERROR";
                    break;
                case Level::WARNING:
                    levelString = "WARNING";
                    break;
                case Level::INFO:
                    levelString = "INFO";
                    break;
                case Level::DEBUG:
                    levelString = "DEBUG";
                    break;
            }

            time_t now = time(nullptr);
            tm *timeinfo = localtime(&now);
            char buffer[80];
            strftime(buffer, sizeof(buffer), "%Y-%m-%d %H:%M:%S", timeinfo);

            logfile << "[" << buffer << "] [" << levelString << "] " << message << std::endl;
            logfile.close();
        }
    }

    // Macros for logging messages with different levels
    void MError(const std::string &message) {
        LogMessage(Level::ERROR, message);
    }

    void MWarn(const std::string &message) {
        LogMessage(Level::WARNING, message);
    }

    void MInfo(const std::string &message) {
        LogMessage(Level::INFO, message);
    }

    void MDebug(const std::string &message) {
        LogMessage(Level::DEBUG, message);
    }
};

int main() {
  University tum;
  Log log(Level::DEBUG);
  log.MDebug("Trying to load the univesity.txt file");
  FileManager::LoadUniversityData(SAVE, tum);
  int choice;

  do {
    system("CLS");
    cout << "TUM Board Menu:\n"
         << "1. Faculty Operations\n"
         << "2. General Operations\n"
         << "0. Exit\n"
         << "Enter your choice: ";
    cin >> choice;
    system("CLS");

    switch (choice) {
      log.MInfo("Selected Faculty Operations: " + to_string(choice));
      case 1: { // Faculty Choice
        int facultyChoice;
        cout << "Faculty Operations:\n"
             << "1. Create and assign a student to a faculty\n"
             << "2. Graduate a student from a faculty\n"
             << "3. Display current enrolled students\n"
             << "4. Display graduates\n"
             << "5. Check if a student belongs to a faculty\n"
             << "0. Return to the Board Menu\n"
             << "Enter your choice: ";
        cin >> facultyChoice;
        system("CLS");

        switch (facultyChoice) { // Faculty choice
          case 1: {
            string first, last, email, enrollment, dob;
            cout << "Enter student first name: ";
            cin >> first;
            cout << "Enter student last name: ";
            cin >> last;
            cout << "Enter student email: ";
            cin >> email;
            cout << "Enter student enrollment date: ";
            cin >> enrollment;
            cout << "Enter student date of birth: ";
            cin >> dob;
            system("CLS");

            Student student(first, last, email, enrollment, dob);

            string abbreviation;
            cout << "Enter the abbreviation of the faculty: ";
            cin >> abbreviation;
            system("CLS");

            bool facultyFound = false;
            for (auto &f : tum.faculties) {
              if (f.GetAbreviation() == abbreviation) {
                f.AddStudent(student);
                cout << "Student added to " << f.GetName() << " faculty.\n";
                log.MInfo("Student added to " + f.GetName() + " faculty.");
                facultyFound = true;


                FileManager::SaveUniversityData(SAVE, tum);
                log.MInfo("Saved the operation in the file.");
                break;
              }
            }

            if (!facultyFound) {
              cout << "Faculty with abbreviation '" << abbreviation << "' not found!\n";
              log.MError("Faculty with abbreviation '" + abbreviation + "' not found when adding a student.");
            Pause();
            break;
            }
            FileManager::SaveUniversityData(SAVE, tum);
            log.MInfo("Saved dada in the file.");
            Pause();
            break;
          }
          case 2: {
            string email;
            system("CLS");
            cout << "Enter student email to graduate: ";
            cin >> email;

            log.MInfo("Attempting to graduate student with email: " + email);

            Faculty *faculty = tum.FindFacultyByStudentEmail(email);
            if (faculty != nullptr) {
              faculty->GraduateStudent(email);
              FileManager::SaveUniversityData(SAVE, tum);
              log.MInfo("Saved dada in the file.");
            } else {
              cout << "Student with email '" << email << "' not found in any faculty.\n";
              log.MWarn("Attempted to graduate student with email '" + email + "', but student not found.");
            }
            Pause();
            break;
          }
          case 3: {
            string abbreviation;
            cout << "Enter faculty abbreviation to display current enrolled students: ";
            cin >> abbreviation;
            tum.DisplayStudentsInFaculty(abbreviation);
            break;
          }
          case 4: {
            string abbreviation;
            cout << "Enter faculty abbreviation to display graduates: ";
            cin >> abbreviation;
            tum.DisplayGraduatesInFaculty(abbreviation);
            break;
          }
          case 5: {
            string email;
            system("CLS");
            cout << "Enter student email to check: ";
            cin >> email;

            log.MInfo("Checking if student with email '" + email + "' belongs to any faculty.");

            Faculty *faculty = tum.FindFacultyByStudentEmail(email);
            if (faculty != nullptr) {
              cout << "Student with email '" << email << "' belongs to " << faculty->GetName() << " faculty, the abbreviation: " << faculty->GetAbreviation();
              log.MInfo("Student with email '" + email + "' belongs to " + faculty->GetName() + " faculty.");
            } else {
              cout << "Student with email '" << email << "' not found in any faculty.\n";
              log.MWarn("Student with email '" + email + "' not found in any faculty.");
            }
            Pause();
            break;
          }
          case 0: { break; }
          default:
            cout << "Invalid choice!\n";
            log.MWarn("Invalid choice entered in faculty operations.");
        }
        break;
      }
      case 2: { // General Choice
        int generalChoice;
        cout << "General Operations:\n"
             << "1. Create a new faculty\n"
             << "2. Search what faculty a student belongs to\n"
             << "3. Display University faculties\n"
             << "4. Display faculties belonging to a field\n"
             << "0. Return to the Board Menu\n"
             << "Enter your choice: ";
        cin >> generalChoice;
        system("CLS");

        switch (generalChoice) { // General Choice
          case 1: {
            string name, abbreviation;
            int field;
            cout << "Enter faculty name: ";
            cin >> name;
            cout << "Enter faculty abbreviation: ";
            cin >> abbreviation;
            cout << "Enter field (0-4): ";
            cin >> field;
            tum.CreateFaculty(name, abbreviation, static_cast<StudyField>(field));
            cout << "Faculty created successfully!\n";
            log.MInfo("Faculty created successfully: " + abbreviation);

            FileManager::SaveUniversityData(SAVE, tum);
            log.MInfo("Saved dada in the file.");
            Pause();
            break;
          }
          case 2: {
            string email;
            system("CLS");
            cout << "Enter student email to search: ";
            cin >> email;

            log.MInfo("Searching faculty for student with email: " + email);

            Faculty *faculty = tum.FindFacultyByStudentEmail(email);
            if (faculty != nullptr) {
              cout << "Student with email '" << email << "' belongs to " << faculty->GetName() << " faculty.\n";
              log.MInfo("Student with email '" + email + "' belongs to " + faculty->GetName() + " faculty.");
            } else {
              cout << "Student with email '" << email << "' not found in any faculty.\n";
              log.MWarn("Student with email '" + email + "' not found in any faculty.");
            }
            Pause();
            break;
          }
          case 3: {
            tum.DisplayFaculties();
            break;
          }
          case 4: {
            int field;
            cout << "Enter the field (0-4): ";
            cin >> field;
            tum.DisplayFacultiesByField(static_cast<StudyField>(field));
            Pause();
            break;
          }
          case 0: { break; }
          default:
            cout << "Invalid choice!\n";
            log.MWarn("Invalid choice entered in general operations.");
        }
        break;
      }
      case 0: // Exiting the Program
        cout << "Exiting...\n";
        FileManager::SaveUniversityData(SAVE, tum);
        log.MInfo("Exiting the program.");
        break;
      default:
        cout << "Invalid choice!\n";
        log.MWarn("Invalid choice entered in main menu.");
    }
  } while (choice);

  return 0;
}
