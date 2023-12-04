import tifffile
import numpy as np
from sklearn.cluster import KMeans
import pandas as pd

# List of image files
image_files = ['2019.tif', '2020.tif', '2021.tif', '2022.tif', '2023.tif']

# Initialize a list to store the results
results = []

# Find the largest dimensions among the provided images
max_height, max_width = 0, 0

for image_file in image_files:
    image = tifffile.imread(image_file)
    height, width = image.shape[:2]
    max_height = max(max_height, height)
    max_width = max(max_width, width)

# Loop through the images
for image_file in image_files:
    # Load the grayscale TIF image using tifffile
    image = tifffile.imread(image_file)

    # Resize the image to the largest dimensions
    resized_image = np.zeros((max_height, max_width))
    resized_image[:image.shape[0], :image.shape[1]] = image

    # Flatten the image to perform k-means clustering
    pixels = resized_image.reshape(-1, 1)

    # Set the number of clusters
    n_clusters = 2  # You can adjust this based on your needs

    # Apply k-means clustering
    kmeans = KMeans(n_clusters=n_clusters, random_state=0).fit(pixels)
    labels = kmeans.labels_

    # Reshape the labels to the original image shape
    clustered_image = labels.reshape(resized_image.shape)

    # Calculate the percentage of each cluster (Intensity: 0 and Intensity: 1)
    percentage_0 = (clustered_image == 0).sum() / clustered_image.size * 100
    percentage_1 = (clustered_image == 1).sum() / clustered_image.size * 100

    # Calculate total water area in square meters
    pixel_size = 10  # meters
    total_water_area_m2 = percentage_1 * (max_height * max_width) * (pixel_size ** 2) / 100

    # Convert total water area to square kilometers
    total_water_area_km2 = total_water_area_m2 / 1e6  # 1 square kilometer = 1e6 square meters

    # Store the results in a dictionary
    result = {
        'Image Name': image_file,
        'Percentage Cluster 0 (Intensity: 0)': percentage_0,
        'Percentage Cluster 1 (Intensity: 1)': percentage_1,
        'Total Water Area (m^2)': total_water_area_m2,
        'Total Water Area (km^2)': total_water_area_km2
    }

    results.append(result)

# Create a DataFrame from the results
df = pd.DataFrame(results)

# Display the results as a table
print(df)
