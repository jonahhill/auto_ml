Properly Formal API Documenation
================================


auto_ml
-------

.. py:class:: Predictor(type_of_estimator, column_descriptions)

  :param type_of_estimator: Whether you want a classifier or regressor
  :type type_of_estimator: 'regressor' or 'classifier'
  :param column_descriptions: A key/value map noting which column is ``'output'``, along with any columns that are ``'nlp'``, ``'date'``, ``'ignore'``, or ``'categorical'``. See below for more details.
  :type column_descriptions: dictionary, where each attribute name represents a column of data in the training data, and each value describes that column as being either ['categorical', 'output', 'nlp', 'date', 'ignore']. Note that 'continuous' data does not need to be labeled as such (all columns are assumed to be continuous unless labeled otherwise).

.. py:method:: ml_predictor.train(raw_training_data, user_input_func=None, optimize_final_model=False, perform_feature_selection=None, verbose=True, ml_for_analytics=True, model_names='GradientBoosting', perform_feature_scaling=True, calibrate_final_model=False, verify_features=False, cv=2)

  :rtype: None. This is purely to fit the entire pipeline to the data. It doesn't return anything- it saves the fitted pipeline as a property of the ``Predictor`` instance.

  :param raw_training_data: The data to train on. See below for more information on formatting of this data.
  :type raw_training_data: DataFrame, or a list of dictionaries, where each dictionary represents a row of data. Each row should have both the training features, and the output value we are trying to predict.

  :param user_input_func: [default- None] A function that you can define that will be called as the first step in the pipeline, for both training and predictions. The function will be passed the entire X dataset. The function must not alter the order or length of the X dataset, and must return the entire X dataset. You can perform any feature engineering you would like in this function. Using this function ensures that you perform the same feature engineering for both training and prediction. For more information, please consult the docs for scikit-learn's ``FunctionTransformer``.
  :type user_input_func: function

  :param ml_for_analytics: [default- True] Whether or not to print out results for which features the trained model found useful. If ``True``, auto_ml will print results that an analyst might find interesting.
  :type ml_for_analytics: Boolean

  :param optimize_final_model: [default- False] Whether or not to perform GridSearchCV on the final model. True increases computation time significantly, but will likely increase accuracy.
  :type optimize_final_model: Boolean

  :param perform_feature_selection: [default- True for large datasets, False for small datasets] Whether or not to run feature selection before training the final model. Feature selection means picking only the most useful features, so we don't confuse the model with too much useless noise. Feature selection typically speeds up computation time by reducing the dimensionality of our dataset, and tends to combat overfitting as well.
  :type perform_feature_selection: Boolean

  :param take_log_of_y: For regression problems, accuracy is sometimes improved by taking the natural log of y values during training, so they all exist on a comparable scale.
  :type take_log_of_y: Boolean

  :param model_names: [default- relevant 'GradientBoosting'] Which model(s) to try. Includes many scikit-learn models, deep learning with Keras/TensorFlow, and Microsoft's LightGBM. Currently available options from scikit-learn are ['ARDRegression', 'AdaBoostClassifier', 'AdaBoostRegressor', 'BayesianRidge', 'ElasticNet', 'ExtraTreesClassifier', 'ExtraTreesRegressor', 'GradientBoostingClassifier', 'GradientBoostingRegressor', 'Lasso', 'LassoLars', 'LinearRegression', 'LogisticRegression', 'MiniBatchKMeans', 'OrthogonalMatchingPursuit', 'PassiveAggressiveClassifier', 'PassiveAggressiveRegressor', 'Perceptron', 'RANSACRegressor', 'RandomForestClassifier', 'RandomForestRegressor', 'Ridge', 'RidgeClassifier', 'SGDClassifier', 'SGDRegressor']. If you have installed XGBoost, LightGBM, or Keras, you can also include ['DeepLearningClassifier', 'DeepLearningRegressor', 'LGBMClassifier', 'LGBMRegressor', 'XGBClassifier', 'XGBRegressor']. By default we choose scikit-learn's 'GradientBoostingRegressor' or 'GradientBoostingClassifier', or if XGBoost is installed, 'XGBRegressor' or 'XGBClassifier'.
  :type model_names: list of strings

  :param verbose: [default- True] I try to give you as much information as possible throughout the process. But if you just want the trained pipeline with less verbose logging, set verbose=False and we'll reduce the amount of logging.

  :param perform_feature_scaling: [default- True] Whether to scale values, roughly to the range of {-1, 1}. Scaling values is highly recommended for deep learning. auto_ml has it's own custom scaler that is relatively robust to outliers.

  :param cv: [default- 2] How many folds of cross-validation to perform. The default of 2 works well for very large datasets. It speeds up training speed, and helps combat overfitting. However, for smaller datasets, cv of 3, or even up to 9, might make more sense, if you're ok with the trade-off in training speed.

  :param calibrate_final_model: [default- False] Whether to calibrate the probability predictions coming from the final trained classifier. Usefulness depends on your scoring metric, and model. The default auto_ml settings mean that the model does not necessarily need to be calibrated. If True, you must pass in values for X_test and y_test as well. This is the dataset we will calibrate the model to. Note that this means you cannot use this as your test dataset once the model has been calibrated to them.

  :param verify_features: [default- False] Allows you to verify that all the same features are present in a prediction dataset as the training datset. False by default because it increases serialized model size by around 1MB, depending on your dataset. In order to check whether a prediction dataset has the same features, invoke ``trained_ml_pipeline.named_steps['final_model'].verify_features(prediction_data)``. Kind of a clunky UI, but a useful feature smashed into the constraints of a sklearn pipeline.


.. py:method:: ml_predictor.predict(prediction_rows)

  :param prediction_rows: A single dictionary, or a DataFrame, or list of dictionaries. For production environments, the code is optimized to run quickly on a single row passed in as a dictionary (taking around 1 millisecond for the entire pipeline). Batched predictions on thousands of rows at a time are generally more efficient if you're getting predictions for a larger dataset.

  :rtype: list of predicted values, of the same length and order as the ``prediction_rows`` passed in. If a single dictionary is passed in, the return value will be the predicted value, not nested in a list (so just a single number or predicted class).


.. py:method:: ml_predictor.predict_proba(prediction_rows)

  :param prediction_rows: Same as for predict above.

  :rtype:  Only works for 'classifier' estimators. Same as above, except each row in the returned list will now itself be a list, of length (number of categories in training data). The items in this row's list will represent the probability of each category.


.. py:method:: ml_predictor.score(X_test, y_test, verbose=2)

  :rtype: number representing the trained estimator's score on the validation data.

  :param verbose: [Default- 2] If 3, even more detailed logging will be included.

.. py:method:: ml_predictor.save(file_name='auto_ml_saved_pipeline.pkl', verbose=True)

  :param file_name: [OPTIONAL] The name of the file you would like the trained pipeline to be saved to.
  :type file_name: string
  :param verbose: If ``True``, will log information about the file, the system this was trained on, and which features to make sure to feed in at prediction time.
  :type verbose: Boolean
  :rtype: the name of the file the trained ml_predictor is saved to. This function will serialize the trained pipeline to disk, so that you can then load it into a production environment and use it to make predictions. The serialized file will likely be several hundred KB or several MB, depending on number of columns in training data and parameters used.
