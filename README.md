DS PROJECT:
QUESTION P:6 
You need to build a manager for all the DA-IICT clubs. The manager ensures that a club member can be looked up in minimum time. A member can either be a faculty or a student. One should be able to search by name, ID, specific club name, or club category (i.e. arts, science& technology, sports, culture). Note that the user of this manager may not be a DA-IICTian and therefore may not know the club’s names:

CODE:

#include <iostream>
#include <cstdio>
#include <fstream>
#include <sstream>
#include <vector>
#include <unordered_map>
#include <algorithm>
#include <string>

using namespace std;

// Define club categories
enum class ClubCategory { 
    Arts, 
    Science, 
    Sports, 
    Culture 
};

// Define a structure for a club
struct Club {
    string name;
    ClubCategory category;

    Club(const string& n, ClubCategory c) {
        this->name = n;
        this->category = c;
    }
};

string toLower(string str) {
    transform(str.begin(), str.end(), str.begin(), ::tolower);
    return str;
}

// Define a structure for a member
struct Member {
    string name;
    int id;
    vector<Club*> clubs;

    Member(const string& n, int i) {
        this->name = n;
        this->id = i;
    }
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
                cerr << "";
                continue;
            }

            // Create member
            Member* member = new Member(name, id);

            // Add member to membersById and membersByName
            membersById[id] = member;
            membersByName[name] = member;

            // Parse clubs
            stringstream clubsSS(clubsStr);
            string clubName;
            while (getline(clubsSS, clubName, ';')) {
                Club* club;
                if (clubsByName.find(clubName) == clubsByName.end()) {
                    // Create new club if it doesn't exist
                    
                    if ((clubName == "PMMC") || (clubName == "Muse") || (clubName == "Music") || (clubName == "Dance")) {
                        club = new Club(clubName, ClubCategory::Arts); // Default category for example
                        clubsByName[clubName] = club;
                        clubsByCategory["Arts"].push_back(club); // Default category for example
                    }
                    if ((clubName == "Research") || (clubName == "Programming") || (clubName == "MSTC") || (clubName == "Business") || (clubName == "Press")) {
                        club = new Club(clubName, ClubCategory::Science); // Default category for example
                        clubsByName[clubName] = club;
                        clubsByCategory["Science"].push_back(club); // Default category for example
                    }
                    if ((clubName == "Cubing") || (clubName == "Chess")) {
                        club = new Club(clubName, ClubCategory::Sports); // Default category for example
                        clubsByName[clubName] = club;
                        clubsByCategory["Sports"].push_back(club); // Default category for example
                    }
                    if ((clubName == "Heritage") || (clubName == "Khelaiya") || (clubName == "DTG")) {
                        club = new Club(clubName, ClubCategory::Culture); // Default category for example
                        clubsByName[clubName] = club;
                        clubsByCategory["Culture"].push_back(club); // Default category for example
                    }

                    
                } else {
                    club = clubsByName[clubName];
                }
                member->clubs.push_back(club);
            }
        }
    }

    // Search for a member by name or ID
  /*  Member* searchMember(const string& query) {
        if (membersByName.find(query) != membersByName.end()) {
            return membersByName[query];
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
    }  */

    // Search for a member by name or ID
// Search for members by name
vector<Member*> searchMember(const string& query) {
    vector<Member*> result;
    for (auto& entry : membersByName) {
        if (entry.first == query) {
            result.push_back(entry.second);
        }
    }

    if (result.empty()) {
        try {
            int id = stoi(query);
            if (membersById.find(id) != membersById.end()) {
                result.push_back(membersById[id]);
            }
        } catch (const invalid_argument& e) {
            cerr << "Invalid query: " << query << endl;
        }
    }

    return result;
}



    // Search for clubs by name
    vector<Club*> searchClub(const string& query) {
        vector<Club*> result;
        string trimmedQuery = query;
        // Trim whitespace from the beginning and end of the query
        trimmedQuery.erase(0, trimmedQuery.find_first_not_of(" \t\n\r\f\v"));
        trimmedQuery.erase(trimmedQuery.find_last_not_of(" \t\n\r\f\v") + 1);

        for (auto& entry : clubsByName) {
            string trimmedName = entry.first;
            // Trim whitespace from the beginning and end of the club name
            trimmedName.erase(0, trimmedName.find_first_not_of(" \t\n\r\f\v"));
            trimmedName.erase(trimmedName.find_last_not_of(" \t\n\r\f\v") + 1);
            if (trimmedName == trimmedQuery) {
                result.push_back(entry.second);
            }
        }

        // Print members of found clubs
        for (Club* club : result) {
            int i = 1;
            cout << endl <<"Members of the " << club->name << " club :" << endl;
            for (auto& entry : membersByName) {
                Member* member = entry.second;
                if (find(member->clubs.begin(), member->clubs.end(), club) != member->clubs.end()) {
                    // cout << " " << i << ") " << member->name << "     ID : " << member->id << endl;
                    printf("%d) %-10s ID : %d\n", i, member->name.c_str(), member->id);
                    i++;
                }
            }
        }

        return result;
    }

    // Search for clubs by category
    vector<Club*> searchClubByCategory(ClubCategory category) {
        vector<Club*> result;
        for (auto& entry : clubsByCategory) {
            if (category == ClubCategory::Arts && entry.first == "Arts") {
                result.insert(result.end(), entry.second.begin(), entry.second.end());
            } else if (category == ClubCategory::Science && entry.first == "Science") {
                result.insert(result.end(), entry.second.begin(), entry.second.end());
            } else if (category == ClubCategory::Sports && entry.first == "Sports") {
                result.insert(result.end(), entry.second.begin(), entry.second.end());
            } else if (category == ClubCategory::Culture && entry.first == "Culture") {
                result.insert(result.end(), entry.second.begin(), entry.second.end());
            }
        }


        // Print clubs and members of found category
        cout << "\nClubs in this category :" << endl << endl;
        int i = 1;
        for (Club* club : result) {
            cout << " " << i << "] " << club->name << " : " << endl ;
            int j = 1;
            for (auto& entry : membersByName) {
                Member* member = entry.second;
                if (find(member->clubs.begin(), member->clubs.end(), club) != member->clubs.end()) {
                    printf("  %d) %-10s ID : %d\n", j, member->name.c_str(), member->id);
                    j++;
                }
            }
            cout << endl;
            i++;
        }
        cout << endl;

        return result;
    }
};

int main() {
    ClubManager manager;
    manager.loadData("data.csv");

    cout << "\n-----> Welcome to DA-IICT Club Community <-----\n";

    string userInput;
    do {
        cout << "\nSEARCH OPTIONS :\n"
             << " 1. Search by name\n"
             << " 2. Search by ID\n"
             << " 3. Search by club name\n"
             << " 4. Search by club category\n"
             << " 5. Exit\n"
             << "\nENTER YOUR CHOICE : ";
        getline(cin, userInput);

        switch (stoi(userInput)) {
            case 1: {
    cout << "\nEnter the Member Name : ";
    getline(cin, userInput);
    vector<Member*> members = manager.searchMember(userInput);
    if (!members.empty()) {
        cout << "\nMembers found with the name " << userInput << ":\n";
        for (Member* member : members) {
            cout << "\nMember Name : " << member->name << "\nID : " << member->id << endl;
            cout << "Clubs : ";
            for (size_t i = 0; i < member->clubs.size(); ++i) {
                cout << member->clubs[i]->name;
                if (i != member->clubs.size() - 1) {
                    cout << ", ";
                }
            }
            cout << endl;
        }
    } else {
        cout << "Member not found." << endl;
    }
    break;
}

            case 2: {
                cout << "\nEnter the ID : ";
                getline(cin, userInput);
                Member* member = manager.searchMember(userInput);
                if (member != nullptr) {
                    cout << "\nMember name : " << member->name << "\nID : " << member->id << endl;
                    cout << "Clubs : ";
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
            case 3: {
                cout << "\nEnter the Club Name : ";
                getline(cin, userInput);
                vector<Club*> clubs = manager.searchClub(userInput);
                if (clubs.empty()) {
                    cout << "\nClub not found." << endl;
                    cout << endl;
                }
                break;
            }
            case 4: {
                cout << "\nEnter the Category (Arts, Science, Sports, Culture) : ";
                getline(cin, userInput);
                ClubCategory category;
                ;
                if (userInput == "Arts") {
                    category = ClubCategory::Arts;
                } else if (userInput == "Science") {
                    category = ClubCategory::Science;
                } else if (userInput == "Sports") {
                    category = ClubCategory::Sports;
                } else if (userInput == "Culture") {
                    category = ClubCategory::Culture;
                } else {
                    cout << "Invalid category." << endl;
                    break;
                }
                vector<Club*> clubs = manager.searchClubByCategory(category);
                if (!clubs.empty()) {
                    cout << "Clubs in the Category " << userInput << " : ";
                    for (size_t i = 0; i < clubs.size(); ++i) {
                        cout << clubs[i]->name;
                        if (i != clubs.size() - 1) {
                            cout << ", ";
                        }
                    }
                    cout << endl;
                } else {
                    cout << "No clubs found in category " << userInput << "." << endl;
                }
                break;
            }
            case 5: {
                cout << "\n\tTHANK YOU!\n\nExiting program... " << endl;
                return 0;
            }
            default:
                cout << "Invalid option. Please try again." << endl;
                break;
        }

    } while (true);

    return 0;
}



   
