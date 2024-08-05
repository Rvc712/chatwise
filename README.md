# chatwise
{
  "_id": ObjectId("user_id"),
  "username": "john_doe",
  "email": "john.doe@example.com",
  "passwordHash": "hashed_password", 
  "fullName": "John Doe",
  "profilePictureUrl": "http://example.com/profile.jpg",
  "bio": "This is a user bio.",
  "createdAt": ISODate("2024-08-05T00:00:00Z"),
  "updatedAt": ISODate("2024-08-05T00:00:00Z"),
  "friends": [ObjectId("friend_id1"), ObjectId("friend_id2")], 
  "pendingFriendRequests": [ObjectId("pending_request_id1")], 
  "receivedFriendRequests": [
    {
      "fromUserId": ObjectId("requester_id"),
      "status": "pending" // Status can be "pending", "accepted", "rejected"
    }
  ]
}
{
  "_id": ObjectId("friend_request_id"),
  "fromUserId": ObjectId("user_id_sender"),
  "toUserId": ObjectId("user_id_receiver"),
  "status": "pending", // Status can be "pending", "accepted", "rejected"
  "createdAt": ISODate("2024-08-05T00:00:00Z"),
  "updatedAt": ISODate("2024-08-05T00:00:00Z")
}
{
  "_id": ObjectId("post_id"),
  "userId": ObjectId("user_id"),
  "content": "This is a text-only post.",
  "createdAt": ISODate("2024-08-05T00:00:00Z"),
  "updatedAt": ISODate("2024-08-05T00:00:00Z"),
  "comments": [
    {
      "commentId": ObjectId("comment_id1"),
      "userId": ObjectId("commenter_user_id"),
      "content": "This is a comment on the post.",
      "createdAt": ISODate("2024-08-05T00:00:00Z"),
      "updatedAt": ISODate("2024-08-05T00:00:00Z")
    }
  ]
}
{
  "_id": ObjectId("comment_id"),
  "postId": ObjectId("post_id"),
  "userId": ObjectId("user_id"),
  "content": "This is a comment on the post.",
  "createdAt": ISODate("2024-08-05T00:00:00Z"),
  "updatedAt": ISODate("2024-08-05T00:00:00Z")
}
create post js
const express = require('express');
const router = express.Router();
const Post = require('./models/Post'); 
const User = require('./models/User'); 

router.get('/feed', async (req, res) => {
    try {
        const userId = req.query.userId; // Get user ID from query parameters


        const user = await User.findById(userId).populate('friends');
        
        if (!user) return res.status(404).send('User not found');

   
        const friendPosts = await Post.find({ userId: { $in: user.friends } });

        // Fetch posts commented by friends
        const commentedPosts = await Post.find({
            comments: { $elemMatch: { userId: { $in: user.friends } } }
        });

   
        const allPosts = [...friendPosts, ...commentedPosts];
        const uniquePosts = Array.from(new Set(allPosts.map(post => post._id)))
                                  .map(id => {
                                      return allPosts.find(post => post._id.toString() === id.toString());
                                  });

        
        uniquePosts.sort((a, b) => b.createdAt - a.createdAt);

        res.json(uniquePosts);
    } catch (err) {
        res.status(500).send(err.message);
    }
});

module.exports = router;
const mongoose = require('mongoose');

const postSchema = new mongoose.Schema({
    userId: { type: mongoose.Schema.Types.ObjectId, ref: 'User' },
    content: String,
    createdAt: Date,
    updatedAt: Date,
    comments: [{
        userId: { type: mongoose.Schema.Types.ObjectId, ref: 'User' },
        content: String,
        createdAt: Date
    }]
});

module.exports = mongoose.model('Post', postSchema);

