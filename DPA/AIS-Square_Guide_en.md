# Tutorial on Contributing DPA Dataset in AIS-Square

## 1. What is DPA？

You can visit [AIS Square｜DPA](https://www.aissquare.com/dpa) to learn more about DPA.

## 2. How to upload to DPA?

Upload your DFT calculation data files using our secure and user-friendly platform.

### 2.1 Prepare DFT calculation data

Collect your DFT calculation result data files and compress them into a single file smaller than 4GB.

> Use supported DFT data formats. The website currently supports all data formats built by **dpdata**. You can view the currently supported data formats on the upload page or [here](https://github.com/deepmodeling/dpdata/blob/master/README.md).

### 2.2 Upload data on the AIS-Square website

To start uploading, open the [AIS-Square website](https://aissquare.com/), click on 【Upload】 in the navigation bar, and select 【Upload Data】. On the upload page, fill in the 【Data Name】 and 【Author】 (don't forget to press Enter to confirm), choose a link address that suits your region (this will affect your upload and download speed), click 【Local Upload】, select the compressed file you created in step 2.1 and confirm. Next:

- If you want to **share your dataset with others** and contribute to the DPA model:

  - Check the box to upload to the DPA dataset, and choose your DFT data format from the dropdown menu
  - Upload a Markdown file to describe your dataset to the public
  - Choose the sharing agreement you need. You can click [here](https://bohrium.dp.tech/) to view all licenses supported by AIS-Square
  - Click 【Publish】, read and agree to the relevant agreements, and officially publish.

- If you prefer **not to make your dataset public**, and only want to contribute to the DPA model.

  - Check the box to upload to the DPA dataset, and choose your DFT data format from the dropdown menu

  - 【Optional】Upload a Markdown file to describe your dataset to the public

  - Check the box to 【Set as Private】, and your dataset will not be made public until you publish a public version later.

    > Please note that others can still download the DPA model trained based on your dataset. Rest assured, this will not reveal your data.

  - Click 【Publish】, read and agree to the relevant agreements, and officially publish.

## 3. Uploaded the dataset

Congratulations! After completing step 2, your dataset has been successfully uploaded to the DPA training server, and we will automatically start the training process. Please be patient and wait.

You can click on the avatar in the upper right corner of the navigation bar to view your uploaded entries in "My Uploads."

Now, let's click on the entry we just uploaded to enter the details page and focus on the training "Progress" on the right side (you can hover your mouse over the "exclamation mark" after each stage in the "Progress" section to see more information). Your dataset will mainly go through the following two stages:

### 3.1 Metric

If you are first in the queue (when you need to wait in line, you will see your position information in the "Queue"), we will "check" the dataset you uploaded to determine if it is suitable to include in the DPA model training:

- Passed: Your uploaded dataset has successfully passed the quality assessment, and it may be a high-quality uncovered dataset. We will create an AIS-Square dataset entry for you and include it in the DPA model training. You will also be a contributor to the DPA pre-trained model. Thank you for your upload!
- Not Passed: Your uploaded dataset did not pass the quality assessment and may have been covered by the DPA training database. However, this dataset may still be a high-quality dataset, and we will create an AIS-Square dataset entry for you. Thank you for your upload!

Regardless of the check results, we will inform you of your data status in the form of pictures. Next, the dataset that passes the quality assessment will enter the training stage →

### 3.2 Training

In this stage, we will include your dataset in the overall pre-training database and train the DPA model together with all existing large-scale data. The training takes about 72 hours, and the training cost will be borne by the AIS-Square official website.

During this process, you can dynamically view the training accuracy of the DPA on your dataset. Please be patient, and when your training task is completed, the system will evaluate the training results. If the results meet the model's growth requirements, we will automatically create a model entry for the DPA version trained by the dataset you contributed and automatically associate your dataset. You will be able to download the DPA model trained by your dataset for free and record the contributors in the DPA pre-trained model project.

## 4. After the training is completed

Once all the processes are completed, we will automatically create an entry for an iterative version of the DPA model based on this training and include it in the "Models" section, with the model entry automatically linked to your dataset. You can also see the model version generated by your dataset in the DPA history versions on the DPA homepage, as well as the RMSE performance of the model version for energy and force on the standard dataset.

## 5. How to use the trained DPA model?

Want to use a trained DPA model? Feel free to check out the [Notebook tutorial](https://nb.bohrium.dp.tech/detail/7145731165) to run DPA models online.