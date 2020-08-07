# Event-Search-Recommendation-Engine
The project aims to use personalization to improve event search and recommendation. Event Search and Recommendation Engine is a full stack project powered by 
TicketMaster API, whose purpose is to recommend events based on user's current geo-location. 

[Project Demo](http://ec2-54-151-66-104.us-west-1.compute.amazonaws.com/Jupiter/)<br/>

![main page](/images/image4.png)<br/><br/><br/>

## Technologies Used
- Java Servlet
- MySQL
- Restful API
- OOD
- AWS
- JavaScript
- Ajax
- HTML/CSS

## Logic Layer of Project
![Logic Layer](/images/image4.png)<br/><br/><br/>

## Current Features
- user can see nearby events based on their current location
- user can like/unlick an event
- System will recommend events to user

## Recommendation Algorithm(content-based)
In this project, I recommend events based on categories that the user has favorited. By knowing the category of the item the user favorited, 
I recommend some events belong to this category nearby this user. Concrete steps are as follow.

1. Fetch all the events (ids) this user has visited.
2. Given all these events, fetch the categories of these events.
3. Given these categories, find what are the events that belong to them.
4. Filter events that this user has visited.
5. Sort the recommendation list on ascending order of distance between recommended events's locations and user's location.

```
public class GeoRecommendation {
	public List<Item> recommendItems(String userId, double lat, double lon) {
		List<Item> recommendedItems = new ArrayList<>();
		DBConnection conn = DBConnectionFactory.getConnection();
		
		// Get all favorite items
		Set<String> favoriteItemIds = conn.getFavoriteItemIds(userId);

		// Get all categories of favorite items, sort by count
		Map<String, Integer> allCategories = new HashMap<>();
		for (String itemId : favoriteItemIds) {
			Set<String> categories = conn.getCategories(itemId);
			for (String category : categories) {
				allCategories.put(category, allCategories.getOrDefault(category, 0) + 1);
			}
		}
		
		List<Entry<String, Integer>> categoryList =
				new ArrayList<Entry<String, Integer>>(allCategories.entrySet());
		Collections.sort(categoryList, new Comparator<Entry<String, Integer>>() {
			@Override
			public int compare(Entry<String, Integer> o1, Entry<String, Integer> o2) {
				return Integer.compare(o2.getValue(), o1.getValue());
			}
		});

		// do search based on category, filter out favorited events, sort by distance
		Set<Item> visitedItems = new HashSet<>();
		
		for (Entry<String, Integer> category : categoryList) {
			List<Item> items = conn.searchItems(lat, lon, category.getKey());
			List<Item> filteredItems = new ArrayList<>();
			for (Item item : items) {
				if (!favoriteItemIds.contains(item.getItemId())
						&& !visitedItems.contains(item)) {
					filteredItems.add(item);
				}
			}
			
			Collections.sort(filteredItems, new Comparator<Item>() {
				@Override
				public int compare(Item item1, Item item2) {
					return Double.compare(item1.getDistance(), item2.getDistance());
				}
			});
			
			visitedItems.addAll(items);
			recommendedItems.addAll(filteredItems);
		}
		
		return recommendedItems;
	  }
}
```

## Install Requirements
- Apache Tomat v9.0
- Eclipse JEE IDE
- JAVA 8
- MySQL
- MAMP

## Future Changes
- The registration system.
- Current project's frontend is built by pure JavaScript without any framework, I will try to refactor it by AugularJS or React in the future.
