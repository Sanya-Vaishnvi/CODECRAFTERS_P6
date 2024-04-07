DS PROJECT:
QUESTION P:6 
You need to build a manager for all the DA-IICT clubs. The manager ensures that a club member can be looked up in minimum time. A member can either be a faculty or a student. One should be able to search by name, ID, specific club name, or club category (i.e. arts, science& technology, sports, culture). Note that the user of this manager may not be a DA-IICTian and therefore may not know the club’s names:

CODE:


#include <iostream>
#include <fstream>
#include <sstream>
#include <vector>
#include <unordered_map>
#include <algorithm>

using namespace std;

// Define club categories
enum class ClubCategory { Arts, ScienceTech, Sports, Culture };

// Define a structure for a club
struct Club {
    string name;
    ClubCategory category;

    Club(const string& n, ClubCategory c) : name(n), category(c) {}
};

// Define a structure for a member
struct Member {
    string name;
    int id;
    vector<Club*> clubs;

    Member(const string& n, int i) : name(n), id(i) {}
};

// Define a manager class to manage clubs and members
class ClubManager {
private:
    unordered_map<string, Club*> clubsByName;
    unordered_map<string, vector<Club*>> clubsByCategory;
    unordered_map<int, Member*> membersById;
    unordered_map<string, Member*> membersByName;

public:
    // Load data from CSV file
    void loadData(const string& filename) {
        ifstream file(filename);
        string line;

        while (getline(file, line)) {
            stringstream ss(line);
            string name, idStr, clubsStr;

            getline(ss, name, ',');
            getline(ss, idStr, ',');
            getline(ss, clubsStr, ',');

            int id;
            try {
                id = stoi(idStr);
            } catch (const invalid_argument& e) {
                cerr << "Invalid ID: " << idStr << endl;
                continue;
            }

            // Create member
            Member* member = new Member(name, id);

            // Add member to membersById and membersByName
            membersById[id] = member;
            transform(name.begin(), name.end(), name.begin(), ::tolower); // Convert name to lowercase
            membersByName[name] = member;

            // Parse clubs
            stringstream clubsSS(clubsStr);
            string clubName;
            while (getline(clubsSS, clubName, ';')) {
                Club* club;
                transform(clubName.begin(), clubName.end(), clubName.begin(), ::tolower); // Convert club name to lowercase
                if (clubsByName.find(clubName) == clubsByName.end()) {
                    // Create new club if it doesn't exist
                    club = new Club(clubName, ClubCategory::Arts); // Default category for example
                    clubsByName[clubName] = club;
                    clubsByCategory["Arts"].push_back(club); // Default category for example
                } else {
                    club = clubsByName[clubName];
                }
                member->clubs.push_back(club);
            }
        }
    }

    // Search for a member by name or ID
    Member* searchMember(const string& query) {
        string queryLower = query;
        transform(queryLower.begin(), queryLower.end(), queryLower.begin(), ::tolower); // Convert query to lowercase

        if (membersByName.find(queryLower) != membersByName.end()) {
            return membersByName[queryLower];
        } else {
            try {
                int id = stoi(query);
                if (membersById.find(id) != membersById.end()) {
                    return membersById[id];
                }
            } catch (const invalid_argument& e) {
                cerr << "Invalid query: " << query << endl;
            }
        }
        return nullptr;
    }

    // Search for clubs by name
    vector<Club*> searchClub(const string& query) {
        vector<Club*> result;
        string queryLower = query;
        transform(queryLower.begin(), queryLower.end(), queryLower.begin(), ::tolower); // Convert query to lowercase

        for (auto& entry : clubsByName) {
            string clubNameLower = entry.first;
            transform(clubNameLower.begin(), clubNameLower.end(), clubNameLower.begin(), ::tolower); // Convert club name to lowercase
            if (clubNameLower == queryLower) {
                result.push_back(entry.second);
            }
        }

        // Print members of found clubs
        for (Club* club : result) {
            cout << "Members of club " << club->name << ":" << endl;
            for (auto& entry : membersByName) {
                Member* member = entry.second;
                if (find(member->clubs.begin(), member->clubs.end(), club) != member->clubs.end()) {
                    cout << member->name << " (ID: " << member->id << ")" << endl;
                }
            }
        }

        return result;
    }

    // Search for clubs by category
    vector<Club*> searchClubByCategory(ClubCategory category) {
        vector<Club*> result;
        for (auto& entry : clubsByCategory) {
            if (entry.first == "Arts" && category == ClubCategory::Arts) { // Modify this condition based on actual categories
                result.insert(result.end(), entry.second.begin(), entry.second.end());
            }
        }

        // Print clubs and members of found category
        cout << "Clubs in this category :" << endl;
        for (Club* club : result) {
            cout << club->name << endl;
            cout << "Members:" << endl;
            for (auto& entry : membersByName) {
                Member* member = entry.second;
                if (find(member->clubs.begin(), member->clubs.end(), club) != member->clubs.end()) {
                    cout << member->name << " (ID: " << member->id << ")" << endl;
                }
            }
            cout << endl;
        }
        cout << endl;

        return result;
    }

};

int main() {
    ClubManager manager;
    manager.loadData("members1.csv");

    string userInput;
    do {
        cout << "Enter search option:\n"
             << "1. Search by name\n"
             << "2. Search by ID\n"
             << "3. Search by club name\n"
             << "4. Search by club category\n"
             << "5. Exit\n"
             << "Option: ";
        getline(cin, userInput);

        switch (stoi(userInput)) {
            case 1: {
                cout << "Enter name: ";
                getline(cin, userInput);
                Member* member = manager.searchMember(userInput);
                if (member != nullptr) {
                    cout << "Member found: " << member->name << " (ID: " << member->id << ")" << endl;
                    cout << "Clubs: ";
                    for (size_t i = 0; i < member->clubs.size(); ++i) {
                        cout << member->clubs[i]->name;
                        if (i != member->clubs.size() - 1) {
                            cout << ", ";
                        }
                    }
                    cout << endl;
                } else {
                    cout << "Member not found." << endl;
                }
                break;
            }
            case 2: {
                cout << "Enter ID: ";
                getline(cin, userInput);
                Member* member = manager.searchMember(userInput);
                if (member != nullptr) {
                    cout << "Member found: " << member->name << " (ID: " << member->id << ")" << endl;
                    cout << "Clubs: ";
                    for



   
