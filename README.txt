mvn package -DskipTests

#####################################
Ch 1 - Meet Hadoop
#####################################

data storage and analysis
	storage capacities of hard drives have increased massively over the years
	but access speeds — the rate at which data can be read from drives — have not kept up
	it takes a long time to read and write data to a single drive

	reduce time by reading from multiple disks at once 
		ex: 100 drives each hold 1/100th of the data
		work in parallel to read the data quickly

	avoid data loss
		many pieces of hardware > greater risk of hardware failure
		avoid data loss through replication

	combine data from many disks
		MapReduce = programming model
			abstracts the problem from disk reads and writes 
			transforms to computation over sets of keys and values

	hadoop provides
		reliable, scalable platform for storage and analysis
		runs on commodity hardware
		open source

	MR = batch query processer
		provides ability to run ad hoc query against entire data set
		get results in a reasonable time
		still, emphasis on batch processing and not for interactive analysis

beyond batch
	Hadoop ecosystem
		no longer just MR and HDFS
		provides infrastructure for distributed computing and large scale data processing
	YARN
		enabler for new process models 
		cluster resource manager system
		allows any distributed program to run data in a hadoop cluster

comparison to relational databases
	disk drive trends
		seek time improving slower vs transfer rate
		seek time
			moving the disk's head to a particular place on disk to read or write data
			latency of disk operation
		transfer rate
			disk's bandwidth = amount of data that can be transmitted in a fix amount of time

	relational database
		update small data set
			traditional B-tree (data structure used for relational DB) works well
		update large data set
			seek data access pattern takes longer to read/write vs streaming 

		database normalization
			retain data integrity
				data doesn't change
			remove redundancy 
				set up tables to for intelligent joins

		good fit for
			point queries or updates 
			database has been indexed to deliver low latency retrieval and update times
			work on relatively small amount of data
			data is continually updated
			working with structured data (with a schema)

	hadoop 
		update large data set
			stream data access pattern = operates on transfer rate = faster read/write vs RDB
			B-Tree less efficient vs MR uses Sort/Merge to rebuild the database

		does not require data normalization
			hadoop does not want to read in nonlocal records in order to perform normalization
			assumes all data needed to create the bigger picture is contained in 1 place

		scales linerally with the size of data
			data is partitioned 
			map and reduce can work in parallel on separate partitions

			original input + cluster size = original speed
			double size of input > job will run 2x as slowly
			but if you double the number of disks > job will run as original speed 

		good fit for
			need to analyze whole dataset in batch fashion
			data is written once and read many times
			working with semi or unstructured data
				designed to interpret data at process time = schema-on-read


###########################
Ch 2 - MapReduce
###########################

data format
	easier and more efficient to process a smaller number of relatively large files

analyzing data using unix tools
	to speed up processing > run program in parallel 
	limitations to manual setup
		dividing work into equal pieces isn't always easy nor obvious
		file sizes differ > some processses finish earlier than others 
		more work = split input into file size chunks and assign each chunk to a process
		combining the results from independent processes requires more work
		limited by processing capacity on a single machine

analyzing data using hadoop
	map and reduce phases requires key - value pair input

	raw input
	0067011990999991950051507004...9999999N9+00001+99999999999... 
	0043011990999991950051512004...9999999N9+00221+99999999999... 
	0043011990999991950051518004...9999999N9-00111+99999999999... 
	0043012650999991949032412004...0500001N9+01111+99999999999...
	0043012650999991949032418004...0500001N9+00781+99999999999...

	mapper input with default key = offset of the beginning of the line from the beginning of the file
	(0, 0067011990999991950051507004...9999999N9+00001+99999999999...)
	(106, 0043011990999991950051512004...9999999N9+00221+99999999999...)
	(212, 0043011990999991950051518004...9999999N9-00111+99999999999...)
	(318, 0043012650999991949032412004...0500001N9+01111+99999999999...)
	(424, 0043012650999991949032418004...0500001N9+00781+99999999999...)

	map = data prep phase
		drop bad records
		pull out year and air temp

	mapper output with key as year and value interpretted as an int
	(1950, 0)
	(1950, 22)
	(1950, −11)
	(1949, 111)
	(1949, 78)

	map output > sort and group by key > reduce input

	reduce input
	(1949, [111, 78])
	(1950, [0, 22, −11])

	reduce function = iterate through the list and pick up the maximum reading

	reduce output
	(1949, 111)
	(1950, 22)

java MR
	Hadoop provides basic types = optimized for network serialization 
		hadoop LongWritable = java Long
		hadoop Text = java String
		hadoop IntWritable = java Integer
		hadoop Context
			allows the Mapper/Reducer to interact with the rest of the Hadoop system
