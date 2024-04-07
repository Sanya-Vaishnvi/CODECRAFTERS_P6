DS PROJECT:
QUESTION P:6 
You need to build a manager for all the DA-IICT clubs. The manager ensures that a club member can be looked up in minimum time. A member can either be a faculty or a student. One should be able to search by name, ID, specific club name, or club category (i.e. arts, science& technology, sports, culture). Note that the user of this manager may not be a DA-IICTian and therefore may not know the club’s names:

CODE:

#include <iostream>
#include <fstream>
#include <sstream>
#include <string>
#include <cstring>
#include <vector>
#include <unordered_map>

using namespace std;

// Forward declarations
class Member;

// Enum for club categories
enum ClubCategory {
    ARTS,
    SCIENCE_AND_TECHNOLOGY,
    SPORTS,
    CULTURE,
};

// Club class
class Club {
private:
    string name;
    ClubCategory category;
    vector<Member*> members;

public:
    Club(const string& name, ClubCategory category) {
        this->name = name;
        this->category = category;
    }

    const string& getName() const {
        return name;
    }

    ClubCategory getCategory() const {
        return category;
    }

    void addMember(Member* member) {
        members.push_back(member);
    }

    const vector<Member*>& getMembers() const {
        return members;
    }
};

// Member class
class Member {
private:
    string name;
    int ID;
    vector<Club*> clubs;

public:
    Member(const string& name, int ID) : name(name), ID(ID) {}

    const string& getName() const {
        return name;
    }

    int getID() const {
        return ID;
    }

    void joinClub(Club* club) {
        clubs.push_back(club);
        club->addMember(this);
    }

    const vector<Club*>& getClubs() const {
        return clubs;
    }
};

// Club Manager class
class ClubManager {
private:
    unordered_map<string, Member*> nameToMember;
    unordered_map<int, Member*> idToMember;
    unordered_map<string, Club*> nameToClub;
    unordered_map<ClubCategory, vector<Club*>> categoryToClubs;

public:
    void loadMembersFromCSV(const string& filename) {
        ifstream file(filename);
        if (!file.is_open()) {
            cerr << "Failed to open " << filename << endl;
            return;
        }

        string line;
        while (getline(file, line)) {
            stringstream ss(line);
            string name;
            int ID;
            string clubName;

            if (getline(ss, name, ',') && (ss >> ID) && getline(ss >> ws, clubName)) {
                // Trim leading and trailing whitespace from clubName
                clubName.erase(0, clubName.find_first_not_of(" \t\n\r\f\v"));
                clubName.erase(clubName.find_last_not_of(" \t\n\r\f\v") + 1);

                addMember(name, ID, clubName);
            }
        }

        file.close();
    }

    void addMember(const string& name, int ID, const string& clubName) {
        Member* newMember = new Member(name, ID);
        nameToMember[name] = newMember;
        idToMember[ID] = newMember;

        // Check if the club already exists
        if (nameToClub.find(clubName) != nameToClub.end()) {
            Club* club = nameToClub[clubName];
            newMember->joinClub(club);
        } else {
            // Create a new club if it doesn't exist
            Club* newClub = new Club(clubName, ARTS); // You may set a default category here
            nameToClub[clubName] = newClub;
            categoryToClubs[newClub->getCategory()].push_back(newClub);
            newMember->joinClub(newClub);
        }
    }

    void printMember(const string& name) {
        if (nameToMember.find(name) != nameToMember.end()) {
            Member* member = nameToMember[name];
            cout << "Member Name: " << member->getName() << endl;
            cout << "Member ID: " << member->getID() << endl;
            cout << "Clubs Joined: " ;
            for (Club* club : member->getClubs()) {
                cout<< club->getName() << endl;
            }
        } else {
            cout << "Member not found!" << endl;
        }
    }

    void printClubJoined(const string& clubName) {
        if (nameToClub.find(clubName) != nameToClub.end()) {
            Club* club = nameToClub[clubName];
            cout << "Club Name: " << club->getName() << endl;
            cout << "Members Joined:" << endl;
            for (Member* member : club->getMembers()) {
                cout << member->getName() << endl;
            }
        } else {
            cout << "Club not found!" << endl;
        }
    }

    void searchByName(const string& name) {
        printMember(name);
    }

    void searchByID(int ID) {
        if (idToMember.find(ID) != idToMember.end()) {
            Member* member = idToMember[ID];
            printMember(member->getName());
        } else {
            cout << "Member not found!" << endl;
        }
    }

    void searchByClubName(const string& clubName) {
        printClubJoined(clubName);
    }

    void searchByCat (const string& cat) {
        if (cat == "Arts") {
            string music = ",music";
            string dance = ",dance";
            searchByClubName(music);
            searchByClubName(dance);
            return;
        }
        else if (cat == "Science") {
            
            return;
        }
        else if (cat == "Sports") {

            return;
        }
        else if (cat == "Culture") {
            
            return;
        }
        else {
            cout << "No clubs of this Category ";
        }
    }

    // Other member and club management functions...

    // Rest of the ClubManager class...
};

int main() {
    ClubManager clubManager;

    // Load members from CSV file
    clubManager.loadMembersFromCSV("members1.csv");

    int choice;
    string name;
    int ID;
    string clubName;

    while (true) {
        cout << "Menu:" << endl;
        cout << "1. Search by Name" << endl;
        cout << "2. Search by ID" << endl;
        cout << "3. Search by Club Name" << endl;
        cout << "4. Search By Category" << endl;
        cout << "5. Exit" << endl;
        cout << "Enter your choice: ";
        cin >> choice;

        string cat;

        switch (choice) {
            case 1:
                cout << "Enter member name: ";
                cin >> name;
                clubManager.searchByName(name);
                break;
            case 2:
                cout << "Enter member ID: ";
                cin >> ID;
                clubManager.searchByID(ID);
                break;
            case 3:
                cout << "Enter club name: ";
                cin >> clubName;
                clubManager.searchByClubName(clubName);
                break;
            case 4:
                cout << "Enter the Category of the Club: ";
                cin >> cat;
                clubManager.searchByCat(cat);
                break;
            case 5:
                cout << "Exiting..." << endl;
                return 0;
            default:
                cout << "Invalid choice! Please enter a number between 1 and 5." << endl;
        }
    }

    return 0;
}



   
