# Split-long-images-into-short-images
Split long images into shorter images while avoiding cutting off text.

调整以下参数避免切割到字体：
`target_height = 2750`  # 目标分割高度
`window_height = 10`  # 文本检测窗口的高度
`text_threshold = 0.001`  # 文本检测阈值
`step = 5`  # 如果检测到文本，向上移动的步长
```python
import os
from PIL import Image
import numpy as np

def binarize_image(image):
    gray = image.convert('L')
    return gray.point(lambda x: 0 if x < 128 else 255, '1')

def detect_text_in_window(binary_image, y, window_height):
    window = binary_image.crop((0, y, binary_image.width, y + window_height))
    window_array = np.array(window)
    black_pixel_percentage = (window_array == 0).mean()
    return black_pixel_percentage

def split_image(image, target_height, window_height, text_threshold, step):
    binary_image = binarize_image(image)
    
    y = 0
    split_positions = []
    
    while y + target_height <= image.height:
        text_percentage = detect_text_in_window(binary_image, y + target_height - window_height, window_height)
        
        if text_percentage < text_threshold:
            split_positions.append(y + target_height)
            y += target_height
        else:
            y += step
    
    split_positions.append(image.height)
    
    split_images = []
    start_y = 0
    for end_y in split_positions:
        split_image = image.crop((0, start_y, image.width, end_y))
        split_images.append(split_image)
        start_y = end_y
    
    return split_images

def process_folder(input_folder, output_folder, target_height, window_height, text_threshold, step):
    if not os.path.exists(output_folder):
        os.makedirs(output_folder)

    for filename in os.listdir(input_folder):
        if filename.lower().endswith(('.png', '.jpg', '.jpeg', '.gif', '.bmp')):
            input_path = os.path.join(input_folder, filename)
            base_name = os.path.splitext(filename)[0]

            try:
                with Image.open(input_path) as img:
                    split_images = split_image(img, target_height, window_height, text_threshold, step)

                for i, split_img in enumerate(split_images):
                    output_filename = f"{base_name}_{i}.jpg"
                    output_path = os.path.join(output_folder, output_filename)
                    split_img.save(output_path, 'JPEG')
                    print(f"Saved: {output_path}")

                print(f"Processed: {filename} - Created {len(split_images)} split images")
            except Exception as e:
                print(f"Error processing {filename}: {str(e)}")

# 可调整的参数
input_folder = r"C:\Users\1\Desktop\lyl1"
output_folder = r"C:\Users\1\Desktop\lyl2"

target_height = 2750  # 目标分割高度
window_height = 10  # 文本检测窗口的高度
text_threshold = 0.001  # 文本检测阈值
step = 5  # 如果检测到文本，向上移动的步长

# 执行文件夹处理
process_folder(input_folder, output_folder, target_height, window_height, text_threshold, step)

print("Folder processing complete.")
```
