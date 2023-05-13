from selenium import webdriver
from selenium.webdriver.common.by import By
import urllib.request
import time
import os

# Set up the webdriver
driver = webdriver.Chrome()

# Navigate to webpage
driver.get("https://www._")

# Create the images folder if it does not exist
if not os.path.exists('images'):
    os.makedirs('images')

# Get the html element
html_element = driver.find_element(By.TAG_NAME, 'html')

# Get all image elements on the webpage
image_elements = driver.find_elements(By.TAG_NAME, 'img')

# Loop through the image elements and download each image
i = 0
while i < len(image_elements):
    # Find the image elements again on each iteration
    html_element = driver.find_element(By.TAG_NAME, 'html')
    image_elements = html_element.find_elements(By.TAG_NAME, 'img')

    # Scroll down to the bottom of the page
    driver.execute_script("window.scrollTo(0, document.body.scrollHeight);")

    # Wait for some time for the page to load new images
    time.sleep(5)

    # Download new images
    for j in range(i, len(image_elements)):
        image_element = image_elements[j]

        # Get the source URL and download the image
        image_url = image_element.get_attribute('src')
        image_name = f'images/image_{j}.jpg'
        urllib.request.urlretrieve(image_url, image_name)

    # Update the counter
    i = len(image_elements)

# Quit the webdriver
driver.quit()
