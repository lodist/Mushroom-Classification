# Mushroom-Classification

This repository contains code for training and applying deep learning models to classify fungi images. The dataset is derived from the 2018 FGVCx Fungi Classification Challenge, hosted on Kaggle, and enriched with private photos and web scraping.

# Strategy
The pipeline for mushroom classification is designed with efficiency and scalability in mind, given the large number of classes (1,394 species) and the diverse dataset. Below is an explanation of the key steps and their strategic importance:

1. Filtering Classes with Insufficient Images
Classes with fewer than 100 images are excluded from the training process to ensure that the models receive a sufficient amount of data for meaningful learning. This threshold helps maintain the quality and reliability of the training process, avoiding overfitting on underrepresented classes.

2. Splitting Classes into Subsets
The dataset is divided into four subsets of classes, each trained on a separate model. This approach addresses the computational limitations of training a single model on all 1,394 classes:
- Reduces GPU memory usage, enabling the pipeline to run on hardware with limited resources.
- Decreases training time for each individual model.
- Enhances accuracy by allowing each model to focus on a smaller, manageable set of classes.
This design ensures the pipeline can scale efficiently for larger datasets or additional classes.

3. TensorFlow Lite Conversion
After training, models are converted to TensorFlow Lite (TFLite) format. This step is critical for deploying the models in mobile and web applications:
- Lightweight and optimized: TFLite models are significantly smaller than standard models, making them ideal for resource-constrained devices.
- Fast inference: Optimizations in TFLite ensure real-time predictions, enabling seamless integration into apps for real-world fungi identification.

4. Ensemble Prediction
The final predictions are generated by combining the results from all four models. This ensemble method aggregates the class probabilities from each model to identify the top predictions:
- Leverages the strengths of individual models specializing in different subsets.
- Enhances the overall accuracy and robustness of the classification process.
- Ensures reliable predictions for even the most challenging cases.

## Ensemble Learning
The ensemble learning process is implemented in the Mushroom_Classification_Ensemble.ipynb notebook. This section explains how the ensemble approach is applied and how it enhances the prediction accuracy:

1. Loading Trained Models:

- The notebook begins by loading the TensorFlow Lite (.tflite) models, which were trained on subsets of the fungi classes using the Mushroom_Classification_Trainer.py script.
- Each model is associated with a specific subset of classes, stored in a .pickle file for reference.

2. Making Predictions:

- For a given input image, each model is used to predict class probabilities for its respective subset.
- These predictions represent the likelihood of the input image belonging to each class in the subset.

3. Aggregating Predictions:
   
The class probabilities from all models are combined into a single set of predictions using the ensemble approach:
- Probabilities for the same class across multiple models are summed to create a combined score.
- The combined scores are normalized to ensure they represent valid probabilities.

4. Identifying Top Predictions:

- After aggregating predictions, the top predicted classes are identified based on their combined probabilities.
- This step ensures that the most likely species of fungi is suggested, taking advantage of the strengths of all models.

5. Improved Accuracy Through Ensemble Learning:

- By aggregating predictions from multiple specialized models, the ensemble approach mitigates biases and inaccuracies that might exist in individual models.
- This method enhances the robustness and reliability of the classification process, particularly for challenging or ambiguous cases.

6. User Interaction:

- The notebook provides a step-by-step guide for users to input their own test images and view the top predictions, including the associated probabilities for each class.

The ensemble learning implementation demonstrates how dividing a large, diverse dataset into manageable subsets and aggregating predictions from specialized models can achieve high accuracy and robustness in mushroom classification tasks.

## Dataset

The dataset contains images from the Danish Svampe Atlas, with over 85,000 training images, 4,000 validation images, and 9,000 testing images. The dataset includes 1,394 fungi species.

### Download the Dataset

The dataset is available on Zenodo. Please download the dataset from the following link:

[Download all_fungi.zip](https://zenodo.org/record/12682745/files/all_fungi.zip)

### Terms of Use

By downloading this dataset you agree to the following terms:
- You will abide by the Danish Svampe Atlas Terms of Service.
- You will use the data only for non-commercial research and educational purposes.
- You will NOT distribute the above images.
- The Danish Svampe Atlas makes no representations or warranties regarding the data, including but not limited to warranties of non-infringement or fitness for a particular purpose.
- You accept full responsibility for your use of the data and shall defend and indemnify the Danish Svampe Atlas, including its employees, officers and agents, against any and all claims arising from your use of the data, including but not limited to your use of any copies of copyrighted images that you may create from the data.

### Quoting the Original GitHub Page

Since the greatest portion of the database stems from Kaggle, follow the following guidelines found here: [https://github.com/visipedia/fgvcx_fungi_comp](https://github.com/visipedia/fgvcx_fungi_comp).

The instructions about data come from that page. Additionally, I have added images of my own, but the majority are from the Danish dataset.

## Prerequisites

Before running the code, ensure you have the following installed:

- Python 3.7 or higher
- TensorFlow
- Scikit-learn
- NumPy
- Matplotlib
- Pandas
- Pickle

## Setup

1. Clone this repository to your local machine:
    ```bash
    git clone https://github.com/lodist/Mushroom-Classification.git
    cd Mushroom-Classification
    ```

2. Install the required Python packages:
    ```bash
    pip install -r requirements.txt
    ```

3. Extract the dataset:
    - Download the dataset
      [Download all_fungi.zip](https://zenodo.org/record/12682745/files/all_fungi.zip)
    - Extract the contents to your repository.

## Script Explanation

The script is provided both in .py as in .ipynb format. The .ipynb format includes ensemble learning to combine predictions from multiple models.

The script performs the following steps:

1. **Imports necessary libraries**: Includes TensorFlow, Scikit-learn, and other libraries for data handling, model building, and deployment.

2. **Configuration parameters**: Centralizes paths and hyperparameters in a CONFIG dictionary for easier adjustment and reuse.

    ```python
    CONFIG = {
        "source_dir": "all_fungi",
        "base_dir": "images",
        "train_dir": "train",
        "val_dir": "val",
        "input_shape": (224, 224, 3),
        "epochs": 30,
        "batch_size": 32,
        "min_images": 100,
        "split_ratio": 0.8
    }
    ```


3. **Move folders with fewer images**: Ensures classes with at least 100 images are included in the training process. Classes with insufficient data are filtered out.

    ```python
    def move_folders_with_fewer_images(source_dir, target_dir, min_images):
        for folder_name in os.listdir(source_dir):
            folder_path = os.path.join(source_dir, folder_name)
            target_folder_path = os.path.join(target_dir, folder_name)
            if os.path.isdir(folder_path):
                image_count = len([
                    file for file in os.listdir(folder_path) 
                    if file.lower().endswith(('jpg', 'jpeg', 'png', 'bmp', 'tiff', 'webp'))
                ])
                if image_count > min_images:
                    shutil.rmtree(target_folder_path, ignore_errors=True)
                    shutil.copytree(folder_path, target_folder_path)
    ```

4. **Prepare and organize data**: Splits the filtered data into training (80%) and validation (20%) sets, ensuring balanced and sufficient data for reliable model training.

    ```python
    def prepare_data(base_dir, train_dir, val_dir, split_ratio):
        supported_extensions = ['.jpg', '.jpeg', '.png', '.ppm', '.bmp', '.pgm', '.tif', '.tiff', '.webp']
        for subdir, _, files in os.walk(base_dir):
            if subdir == base_dir:
                continue
            image_files = [f for f in files if any(f.lower().endswith(ext) for ext in supported_extensions)]
            if not image_files:
                continue
            train_files, val_files = train_test_split(image_files, train_size=split_ratio, random_state=42)
            class_name = os.path.basename(subdir)
            train_class_dir = os.path.join(train_dir, class_name)
            val_class_dir = os.path.join(val_dir, class_name)
            os.makedirs(train_class_dir, exist_ok=True)
            os.makedirs(val_class_dir, exist_ok=True)
            for file_name in train_files:
                shutil.copy(os.path.join(subdir, file_name), os.path.join(train_class_dir, file_name))
            for file_name in val_files:
                shutil.copy(os.path.join(subdir, file_name), os.path.join(val_class_dir, file_name))
    ```

5. **Split class names into subsets**: The class names are split into four subsets to manage the computational complexity of training a single model with 1,394 classes. This approach allows each model to focus on a manageable subset of classes, reducing memory usage and training time while improving accuracy for fine-grained classification tasks.

    ```python
    class_names = sorted(os.listdir(CONFIG["train_dir"]))
    split_size = len(class_names) // 4
    class_names_split = [class_names[i:i + split_size] for i in range(0, len(class_names), split_size)]
    if len(class_names_split) > 4:
        class_names_split[-2].extend(class_names_split[-1])
        class_names_split = class_names_split[:-1]
    ```

6. **Train models for each subset**: Trains four separate models, each on a subset of the classes, to handle the large dataset efficiently.

    ```python
    for i, class_subset in enumerate(class_names_split):
        print(f"Training model for class subset {i + 1}")
        train_loader, val_loader = get_data_loaders_subset(CONFIG["train_dir"], CONFIG["val_dir"], class_subset)
        model = build_model(CONFIG["input_shape"], len(class_subset))
        model_path = f'mushroom_classification_model_{i}.keras'
        train_model(model, train_loader, val_loader, model_path, CONFIG["epochs"])
        model.save(model_path)
    ```

7. **Convert models to TensorFlow Lite**: Converts the trained models to TensorFlow Lite format for efficient deployment on resource-constrained devices, such as mobile phones or edge devices. TensorFlow Lite optimizes model size and inference speed, making it ideal for real-time fungi identification applications.

    ```python
    for model_path in model_paths:
        convert_to_tflite(model_path)
    ```

8. **Apply models to make predictions**: Predictions are made using all trained models, each specializing in a subset of classes. The results from these models are combined using an ensemble approach, where probabilities are aggregated to identify the top predictions.

    ```python
    top_3_predictions = predict_ensemble(image_path, model_paths, 'class_names_split.pickle')
    ```

## Results

The results demonstrate the effectiveness of the class-splitting and ensemble strategy, achieving a training accuracy of 91.81% and validation accuracy of 91.32%. This performance highlights the robustness of the approach in handling a large and diverse dataset, making it suitable for practical applications in fungi classification. 
The training loss and validation loss were recorded at 0.2546 and 0.3142 respectively.  
By splitting the dataset into smaller subsets and training four separate models, each handling a portion of the classes, we achieved efficient model training and reduced computational load.  
This approach ensures that each model focuses on a manageable number of classes, leading to better generalization and improved accuracy.  
Additionally, combining the predictions from these smaller models provides a robust ensemble method, enhancing overall prediction accuracy and reliability.


![output](https://github.com/lodist/Mushroom-Classification/assets/75701170/6d8d6b99-39bb-4f13-9d45-466a7d6e6a73)

Final Training Accuracy: 0.9181  
Final Validation Accuracy: 0.9132  
Final Training Loss: 0.2546  
Final Validation Loss: 0.3142

## Download Models
The models used in this project are provided in TensorFlow Lite (.tflite) format and can be found in the /models folder. 


## License
This script is open-source and can be used by anyone under the MIT License.
