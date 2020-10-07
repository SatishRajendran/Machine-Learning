# Data Preprocessing (Machine Learning - Matlab)
%%Data Pre-Processing Template
%% Main Function
global Column_names;
%Function to read the data
data = get_data();
%Function to either remove or fill the missing data
data = remove_fill_missing_data(data);
%Function to Scale the features
data = Scaled_Features(data);
%Fucntion to encode the categorical data
data = Encoded_data(data);

%% Function to read the raw data
function data = get_data()
    flag = true;
    while(flag)
        %try-catch block to handle the error
        try
            %Input the name of the data file
            x = input("Enter the name of the data file  :","s");
            data = readtable(x);
            flag = false;
        catch me
            fprintf(" Following error occured %s\n",me.message);
            flag = true;
        end
     end
    
end
%% Function to remove or fill the misising data
function data = remove_fill_missing_data(x)
    cflag = true;
    while cflag
        flag = true;
        while(flag)  
            t = input("Enter the command either remove or fill the misssing data :","s");
            if t == "remove" || t == "fill"
                flag = false;
            else
                disp("Invalid Operation.Please specify the command as listed above");
                flag = true;
            end
        end 
        if t == "remove"
            aflag = true;
            while aflag
                disp(["Choose the removal operation to perform\n"
                "1.rerow(Remove Entire row with NaN Value)\n"
                "2.recol(Remove Entire Column with NaN Value)\n"
                "3.relr(Remove  row  based on Relative Percentage of missing values)\n"
                "4.relc(Remove  column based on Relative Percentage of missing values)"]);
                y = input("Enter the option of your interest  :","s");
                if y == "rerow"
                    x = rmmissing(x);
                    aflag = false;
                elseif y == "recol"
                    x = rmmissing(x,2);
                    aflag = false;
                elseif y== "relr"
                    m = input("Entire the minimun number of NaN values to be present for removal");
                    x = rmmissing(x,"MinNumMissing",m);
                    aflag = false;
                elseif y== "relc"
                    m = input("Entire the minimun number of NaN values to be present for removal");
                    x = rmmissing(x,2,"MinNumMissing",m);
                    aflag = false;
                else
                    disp("Invalid Input.Try again!!!!!");
                    aflag = true;
                end
            end
            
%% Command to check whether the data still contains NaN Values
            if any (sum(ismissing(x)) >= 1)
                disp("You still have some missing values.Would you like to perform operation again");
                w = input("Yes or No  :","s");
                if w == "Yes"
                    cflag = true;
                else
                    cflag = false;
                    data = x;
                end
            else 
                data = x;
                cflag = false;
            end
         elseif t == "fill"
            global Column_names;
            flag = true;
            while(flag)
                %try-catch block to handle the error
                try
                    disp(["Choose the operation to fill the missing data\n"
                          "1.mean\n"
                          "2.median\n"
                          "3.previous\n"
                          "4.next"]);
                    y = input("Enter the name of operation to perform  ","s");
                    if y == "mean" || y == "median" || y == "previous" || y == "next"
                        flag = false;
                    end
                catch
                    disp("Invalid Operation.Please Specify the Operation listed above ");
                    flag = true;
                end
             end
            Column_names = x.Properties.VariableNames;
            for i = 1:length(Column_names)
                %Check whether the column is numeric
                if isnumeric(x.(Column_names{i}))
                    if y == "mean"
                       x.(Column_names{i})=fillmissing(x.(Column_names{i}),"constant",mean(x.(Column_names{i}),"omitnan"));
                    elseif y == "previous"
                       if isnan(x.(Column_names{i})(1))
                           x.(Column_names{i})(1)= x.(Column_names{i})(2);
                           x.(Column_names{i})(2:end)=fillmissing(x.(Column_names{i})(2:end),"previous");
                       else
                           x.(Column_names{i})=fillmissing(x.(Column_names{i}),"previous");
                       end
                     elseif y=="median"
                        x.(Column_names{i})=fillmissing(x.(Column_names{i}),"constant",median(x.(Column_names{i}),"omitnan"));
                     elseif y=="next"
                        if isnan(x.(Column_names{i})(end))
                           x.(Column_names{i})(end)= x.(Column_names{i})(end-1);
                           x.(Column_names{i})(1:end-1)=fillmissing(x.(Column_names{i})(1:end-1),"next");
                         else
                           x.(Column_names{i})=fillmissing(x.(Column_names{i}),"next");
                        end
                    end
                else
                      x.(Column_names{i})=fillmissing(x.(Column_names{i}),"constant",cellstr(mode(categorical(x.(Column_names{i})))));
                end
            end
            data = x;
            cflag = false;
        end
    end
end
%% Scale the features
function data = Scaled_Features(x)
    global Column_names;
    flag = true;
    while(flag) 
       y = input("Enter the name of type of Feature Scaling either Standardisation or Normalisation  :","s");
            if y == "Standardisation" || y == "Normalisation"
                for i = 1:length(Column_names) 
                    %Check whether the column is numeric
                    if isnumeric(x.(Column_names{i}))
                        if y == "Standardisation" 
                            x.((Column_names{i})) = (x.((Column_names{i})) - mean(x.((Column_names{i})))) / std(x.((Column_names{i})));
                        elseif y == "Normalisation"
                            x.((Column_names{i})) =  (x.((Column_names{i})) - min( x.((Column_names{i})))) / (max(x.((Column_names{i})))-min(x.((Column_names{i}))));
                        end
                    end
                end
                data = x;
                flag = false;
            else
                disp("Invalid Operation");
                flag = true;
            end
     end
 end
         
%% Encode the categorical data
 function data = Encoded_data(x)
     global Column_names;
     for i = 1:length(Column_names)
        if ~isnumeric(x.(Column_names{i}))
            name = Column_names{i};
            %Get the unique Values from the Column
            unique_value = unique(x.(Column_names{i}));
            if length(unique_value) >= 3
                for a = 1:length(unique_value)
                    table_name(:,a) = double(ismember(x.(Column_names{i}),unique_value(a)));
                end
                [r , c] = size(table_name);
                x.(name) = [];
                %Loop to create a table
                for j = 1:c
                    t = table(table_name(:,j),"VariableNames",unique_value(j));
                end
                data = [x t];
                x = data;
             else    
                for b = 1:length(unique_value)
                    table_name = double(ismember(x.(Column_names{i}),unique_value(b)));
                end
                x.(name) = [];
                [r , c] = size(table_name);
                for j = 1:c
                    t = table(table_name(:,j),'VariableNames',cellstr(name));
                end
                data = [x t];
                x = data;
            end
        end
     end
     disp("Succcessfully Completed data_preprocessing");
 end
