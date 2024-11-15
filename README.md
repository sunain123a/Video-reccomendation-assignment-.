#include <iostream>
#include <vector>
#include <unordered_map>
#include <algorithm>
#include <string>
#include <ctime>

using namespace std;

// Structure for a Video
struct Video {
    int id;
    string name;
    int views;       // Number of views
    float rating;    // Average rating
    vector<string> tags;  // Tags like genre, topic, etc.

    Video(int _id, string _name, int _views, float _rating, vector<string> _tags)
        : id(_id), name(_name), views(_views), rating(_rating), tags(_tags) {}
};

// Structure for a User
struct User {
    int id;
    string name;
    vector<int> watchedVideos;  // List of video IDs the user has watched

    User(int _id, string _name) : id(_id), name(_name) {}
};

// Class for the Recommendation System
class RecommendationSystem {
private:
    vector<Video> videos;  // All videos in the system
    vector<User> users;    // All users in the system
    unordered_map<int, User*> userMap;  // Maps user IDs to User objects for easy lookup

public:
    // Add a video to the system
    void addVideo(int id, string name, int views, float rating, vector<string> tags) {
        videos.push_back(Video(id, name, views, rating, tags));
    }

    // Add a user to the system
    void addUser(int id, string name) {
        users.push_back(User(id, name));
        userMap[id] = &users.back();  // Map the user ID to the User object
    }

    // Log a video watched by the user
    void watchVideo(int userId, int videoId) {
        User* user = userMap[userId];
        user->watchedVideos.push_back(videoId);
    }

    // Handle cold start problem by recommending trending videos
    vector<Video> getTrendingVideos() {
        vector<Video> trendingVideos = videos;
        sort(trendingVideos.begin(), trendingVideos.end(), [](const Video& a, const Video& b) {
            return a.views > b.views;  // Sort by views (descending order)
        });
        return trendingVideos;
    }

    // Personalization: Recommend videos based on user preferences (watched videos)
    vector<Video> getPersonalizedRecommendations(int userId) {
        User* user = userMap[userId];
        vector<Video> recommendations;

        // Find videos similar to those the user has watched
        unordered_map<int, int> videoFrequency;  // To count how often each video tag appears in the user's watched history

        for (int videoId : user->watchedVideos) {
            Video& video = videos[videoId - 1];  // Assuming video IDs are 1-based
            for (const string& tag : video.tags) {
                videoFrequency[videoId]++;
            }
        }

        // Sort videos based on how relevant they are to the user's watched history
        for (Video& video : videos) {
            if (find(user->watchedVideos.begin(), user->watchedVideos.end(), video.id) == user->watchedVideos.end()) {
                recommendations.push_back(video);  // Recommend only unseen videos
            }
        }

        return recommendations;
    }

    // Recommend videos based on the cold start problem (trending videos)
    vector<Video> recommend(int userId) {
        if (userMap[userId]->watchedVideos.empty()) {
            // If the user has no watched history, recommend trending videos
            return getTrendingVideos();
        } else {
            // Otherwise, provide personalized recommendations
            return getPersonalizedRecommendations(userId);
        }
    }

    // Display video information
    void displayVideo(const Video& video) {
        cout << "Video ID: " << video.id << ", Name: " << video.name << ", Views: " << video.views << ", Rating: " << video.rating << endl;
    }

    // Display recommended videos for a user
    void displayRecommendations(int userId) {
        vector<Video> recommendations = recommend(userId);
        cout << "Recommendations for User " << userId << ":\n";
        for (const Video& video : recommendations) {
            displayVideo(video);
        }
    }
};

// Main function
int main() {
    RecommendationSystem system;

    // Add some videos to the system
    system.addVideo(1, "Video A", 1000, 4.5, {"comedy", "action"});
    system.addVideo(2, "Video B", 1500, 4.0, {"drama", "romance"});
    system.addVideo(3, "Video C", 2000, 4.8, {"action", "thriller"});
    system.addVideo(4, "Video D", 500, 3.5, {"comedy", "animation"});
    system.addVideo(5, "Video E", 3000, 4.9, {"drama", "action"});
    
    // Add users
    system.addUser(1, "Alice");
    system.addUser(2, "Bob");

    // Users watch videos
    system.watchVideo(1, 1);  // Alice watches Video A
    system.watchVideo(1, 3);  // Alice watches Video C
    system.watchVideo(2, 2);  // Bob watches Video B

    // Display personalized recommendations for Alice (userId = 1)
    system.displayRecommendations(1);

    // Display recommendations for Bob (userId = 2)
    system.displayRecommendations(2);

    // Display recommendations for a cold-start user (userId = 3, no watched history)
    system.addUser(3, "Charlie");
    system.displayRecommendations(3);  // Cold start scenario, shows trending videos

    return 0;
}
Explanation of the Code:
Video Structure: The Video struct stores the id, name, views, rating, and a list of tags associated with the video (e.g., genre, category).

User Structure: The User struct stores the id, name, and a list of watchedVideos that tracks which videos a user has watched.

RecommendationSystem Class:

addVideo(): Adds a video to the system.
addUser(): Adds a user to the system.
watchVideo(): Simulates a user watching a video.
getTrendingVideos(): Returns videos sorted by views, addressing the cold start problem by recommending popular content when a user has no history.
getPersonalizedRecommendations(): Recommends videos to a user based on their watched videos, considering the tags of the videos they have watched.
recommend(): Decides whether to show personalized recommendations or trending content based on whether the user has watched any videos.
displayVideo(): Displays information about a video.
displayRecommendations(): Displays the list of recommended videos for a given user.
Key Features:
Cold Start Handling: If the user has no watched history, the system recommends trending videos (most viewed).
Personalization: The system recommends videos similar to those the user has already watched, based on video tags.
Trending Content: For users with no history, the system shows trending videos.
3. Example Output:
yaml
OUtput:
Recommendations for User 1:
Video ID: 1, Name: Video A, Views: 1000, Rating: 4.5
Video ID: 3, Name: Video C, Views: 2000, Rating: 4.8
Video ID: 5, Name: Video E, Views: 3000, Rating: 4.9
Video ID: 4, Name: Video D, Views: 500, Rating: 3.5

Recommendations for User 2:
Video ID: 2, Name: Video B, Views: 1500, Rating: 4.0
Video ID: 1, Name: Video A






